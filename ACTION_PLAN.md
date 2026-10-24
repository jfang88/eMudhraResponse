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


## Remediation without a full application upgrade

**Policy:** require demonstrated rejection of the compromised chains, not migration to the newest application or runtime major version. Prefer a supported trust-data fix or maintenance-branch security update when it covers the application's actual verifier. Keep unrelated lifecycle/security findings separate; incident mitigation does not make unsupported software generally safe.

### Verified delivery options (24 October 2026)

| Platform | Option that can avoid a full application upgrade | Conditions / limitations |
| --- | --- | --- |
| Ubuntu | Targeted updates to ca-certificates and libnss3 rather than upgrading all packages. [R1] | Only covers consumers of those components. Canonical documents an additional Java-store synchronisation issue; preserve locally provisioned CAs when following its workaround. Snap apps have separate update paths. |
| Windows | Microsoft trusted/disallowed Certificate Trust List updates; supported internal distribution for disconnected environments. [R2] | Trust-data delivery, not a universal application installer. Prove effective distrust under the service identity; private Java/PEM/application stores are separate. |
| Azul Zulu | Vendor-provided replacement blocklist file for existing installations, with platform-specific downloads and checksums. [R3] | Follow the exact Zulu/version/platform guidance; replace the correct blocked.certs or blacklisted.certs file, preserve permissions and restart all consuming JVMs. Do not assume it is supported on other JDK vendors. |
| Oracle JDK | Vendor-documented removal of relevant roots from the effective keystore, or enabled revocation checking. [R4] | Oracle's 23 October advisory requires additional action even for listed updated builds. Apply to each installation/custom store as applicable. Root removal can affect other services; revocation requires validation of network reachability and failure behaviour. |
| Python using certifi | Update only the CA-bundle dependency to a compatible fixed release, or configure the HTTP client to use a separately managed bundle where supported. [R5][R6] | No need to assume all application dependencies must change. Certifi does not support editing its packaged bundle in place. Requests supports an explicit verify path or REQUESTS_CA_BUNDLE, but confirm the actual call path honours it. |
| Embedded roots / custom verifiers / vendor appliances | Vendor hotfix, supported maintenance release, or a documented external-store configuration if available. | If the verifier cannot consume external trust/distrust policy, a store-only fix may not exist; a runtime rebuild or vendor update can be necessary. Obtain product-specific confirmation. |
| Containers | Rebuild the existing application release with updated trust data, or deploy a versioned managed trust volume when the application supports it. | Preserve application version where feasible. Changes must survive restart and scale-out; confirm the actual workload reads the changed store. |

These are distinct mechanisms: package update, data-file replacement, trust-list distribution and configuration change. There is no single verified standalone installer that fixes every enterprise verifier.

### Choose the smallest effective change

1. Identify the actual verifier, effective stores and vendor support constraints.
2. Prefer an applicable vendor trust-data/blocklist update or targeted package fix.
3. If unavailable, assess a documented external trust-store override or carefully controlled trust-policy change.
4. If those paths cannot block the affected chains, request a supported hotfix/backport or maintenance release. Escalate an exception if none is available.
5. Record support status and validate both emSign and AAA cross-sign routes, normal business connections and persistence after restart/redeployment.

### Rules for managed trust-store changes

- **Deletion is not distrust:** deleting an intermediate from a local cache/store does not block a peer from supplying it again. Use an enforced distrust mechanism or remove all relevant trust-anchor paths.
- **An additive bundle is insufficient:** for example, NODE_EXTRA_CA_CERTS adds trust; it does not subtract an already trusted built-in CA. Select a documented replacement mechanism for the actual Node version/client where available. [R7]
- **Plain PEM root bundles normally do not express a portable intermediate denylist.** A broad anchor removal may be the available remedy, but it can break legitimate chains. Apply vendor-specific instructions; do not delete roots by a loose subject-name match.
- **Revocation is conditional:** confirm intermediate checks are active, required CRL/OCSP endpoints are reachable and unavailable-status behaviour is acceptable. Merely setting a flag is not completion.
- Version the resulting trust data and record its source, hashes, exact certificate identities and owner. Preserve required private enterprise CAs. Make the change repeatable in images and configuration management so reinstalls do not undo it.
- Retain backups for recovery, but do not silently restore vulnerable trust to resolve an outage. Address legitimate chain issues or use an approved compensating control.
- A custom verifier can ignore all these settings. Validate through the real application client before accepting a store-only remedy.

### Additional completion evidence

Record the remediation method (trust-data / targeted package / configuration / runtime patch), retained application version, vendor applicability, modified store/blocklist hash, restart/deployment record, emSign and AAA test results, legitimate-service tests and drift-control ownership. Custom temporary remedies also need an expiry/review date.

### Remediation references

- [R1: Canonical targeted fixes and Java-store caveat](https://ubuntu.com/blog/emudhra-ca-compromise-fixes-available).
- [R2: Microsoft CTL and disallowed-certificate distribution](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates).
- [R3: Azul downloadable blocklist workaround](https://docs.azul.com/core/emudhra-workaround).
- [R4: Oracle JDK eMudhra guidance, updated 23 October](https://ops.java/emudhra).
- [R5: Certifi bundle and modification policy](https://github.com/certifi/python-certifi).
- [R6: Requests alternate CA-bundle configuration](https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification).
- [R7: Node.js CA-selection configuration](https://nodejs.org/api/cli.html).

These options are a decision framework, not instructions already applied to the enterprise. Recheck the exact vendor advisory before implementation.

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
