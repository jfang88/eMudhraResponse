# Enterprise action plan

**24 October 2026 | Proposed targets measured from response activation (T0)**

## Ownership and distribution

Security should establish one parent incident/change record and route the work packages below to the corresponding service-management queues. Assign a named owner and deputy, obtain acknowledgement, and link all evidence to the parent record. Distribution and implementation are pending; this repository does not identify individual recipients.

| Work package / recipient | Required action | Closure evidence |
| --- | --- | --- |
| SEC — Security response lead | Set scope and priority; coordinate assessment; review exceptions and potential exploitation with SOC. Maintain the asset denominator and unresolved findings. | Daily consolidated report; owner for every affected or unassessed service; risk decisions recorded. |
| EUC — Desktop / browser team | Check managed browser security-update delivery and policies; update OS and affected desktop apps, including bundled Electron runtimes. Include developer desktops, VDI and infrequently connected laptops. | Fleet compliance export plus sampled rejection tests for each distinct platform/browser/app configuration; stale/offline devices tracked separately. |
| LNX — Linux server team | Apply vendor CA-package and software fixes. Inventory independent bundles, runtime stores, service overrides and images used to provision servers. Coordinate application restarts. | Fleet package evidence and application-linked validation; custom stores and disconnected servers accounted for. |
| WIN — Windows server team | Confirm vendor distrust updates actually reach supported servers, including restricted-network systems. Assess service-specific stores, Java/Python/Node installations and bundled applications independently. | Effective trust-policy evidence under the service identity, runtime inventory and relevant rejection tests; reboot/restart records where needed. |
| APP — Application / middleware owners | Map every distinct TLS verifier and outbound client; inspect custom stores and verification overrides. Execute the validation runbook with the deployed library, configuration and service identity. | Store provenance, fix/deployment evidence, negative tests and successful legitimate integration tests for each verification path. |
| CTR — Container platform / DevSecOps | Inventory running image digests and dependencies. Scan OS packages, runtimes and copied CA bundles; rebuild and redeploy affected images. Include init containers, sidecars, jobs and build agents. | SBOM/scan coverage, new digests mapped to workloads, mounted-store checks and tests through the application inside its deployed environment. |
| NET — Network / proxy / WAF / PKI teams | Assess certificate verification on outbound proxy connections and backend TLS legs. Inspect relevant mTLS trust policies and affected server certificate chains. Separate frontend, backend and client-authentication decisions. | Vendor guidance, effective configuration and tests per TLS leg; affected server certificate replacements and functional checks. |
| VND — Supplier / service owners | Request product-specific applicability and remediation statements, including appliances and managed services. | Deployed product/version evidence, supplier response and deadline or compensating control for outstanding fixes. |
| SOC — Detection / investigation | Review available TLS, proxy, endpoint and application telemetry for anomalies and incident-relevant certificate indicators. Escalate evidence of impersonation or credential exposure. | Documented review scope and retention limits; investigation records where warranted. Absence of alerts is not proof of no exploitation. |

## Suggested milestones

- **T0 to 24 hours:** name owners; acknowledge tasks; identify critical services, trust paths and unsupported assets; establish baseline coverage. Escalate confirmed vulnerable critical paths immediately.
- **Within 48 hours:** target critical-path remediation and validation, or document compensating controls and a named risk owner.
- **Within seven days:** target remaining supported estate; review each unresolved or unavailable vendor fix with an expiry date.
- **Daily until closure:** report assessed/total, patched/affected, validated/affected, unassessed, failed tests, service disruptions and overdue exceptions. These are proposed operational targets, not vendor-mandated deadlines.

## Container scanning and patching workflow

1. Enumerate production, DR, scheduled and scale-to-zero workloads and their immutable image digests. Include CI runners, image builders and deployment tooling.
2. Refresh scanner advisories and generate or inspect SBOMs. Compare relevant software with vendor fixes linked by the [maintained package list](https://emudhra.we-have-a-problem.com/#developers).
3. Inspect final image filesystems and configuration for manually copied trust bundles, embedded roots and mounted stores. A package scan can miss unmanaged files, static binaries and application-specific trust.
4. Fix source Dockerfiles/build configuration and base-image references; rebuild with refreshed dependencies and CA data. Do not rely on installing a patch into an ephemeral running container.
5. Deploy new digests and restart affected processes. Confirm the old replicas are gone and mounts/init scripts have not restored stale stores. A debug sidecar's curl test proves only that sidecar's verifier.
6. Test the application path, plus normal integrations. Address old rollback images and deployment templates so a rollback or scale-out cannot silently restore exposure.

## Additional work packages: certificate inventory and alternate trust paths

- **DevSecOps with PKI:** retain Trivy or Syft/Grype for package analysis; pilot CBOMkit-theia or an equivalent certificate collector. Do not treat ordinary CycloneDX SBOM output as a complete CBOM. Use the [tooling guide](CBOM_AND_CROSS_SIGNING.md).
- **Application, Linux and Windows owners:** collect effective stores separately from package inventories. Record mounted/overridden stores, unsupported formats and active distrust policies. Join certificate and SPKI matches to actual application identities.
- **PKI and Security:** maintain dated affected-certificate/key references, including Sectigo AAA cross-signs. An AAA anchor present in a store is a triage signal, not proof of vulnerability. Require both emSign and AAA-route tests before closure.
- **Endpoint security / software distribution:** assess code-signing validation separately from TLS, including relevant platform protections and supplier evidence. Assign a named owner under EUC or APP; document non-applicability where justified.
- **Response lead:** report certificate-inventory coverage and runtime validation separately. Do not defer urgent vendor remediation while selecting a CBOM tool.

Sectigo reports revocation of the affected cross-certificates and no compromise of its own systems or keys. Its [advisory](https://www.sectigo.com/resource-library/advisory-on-compromised-emudhra-intermediate-certificates) does not remove the need to validate clients that may not consume revocation information.

## Closure and exceptions

Close a work package only when the inventoried scope is complete, fixes are active, affected chains are rejected by the intended verifier, and legitimate connections still work. Sampling requires a documented equivalence group (same image/runtime, configuration, identity and trust policy); do not extrapolate across different stacks.

Retain exceptions for unknown, offline, unsupported, blocked-from-testing and vendor-pending systems. Record the exposure, compensating control, accountable risk owner, expiry and retest date. A proxy mitigation can protect a specific route without fixing the underlying application's trust store.

## Ready-to-distribute task brief

**Subject:** Action required — eMudhra certificate trust assessment and remediation

**To:** Named owner of the applicable work package above

Please acknowledge ownership within one working day. Inventory the systems and application verification paths in your scope, apply applicable vendor fixes, validate rejection of affected certificate chains, and confirm normal service operation. Return the [completion record](COMPLETION_TRACKER.md) with evidence links and unresolved exceptions. Do not report completion based solely on patch installation, a browser result or a clean vulnerability scan. Escalate service disruption and confirmed exposure immediately.

This brief is prepared for internal routing; it has not been sent.

## Technical basis

[Cloudflare guidance](https://blog.cloudflare.com/emudhra-certificate-compromise/) and the [incident reference](https://emudhra.we-have-a-problem.com/) support the need to assess non-browser verification paths. The ownership model, milestones and evidence requirements above are proposed enterprise implementation controls.
