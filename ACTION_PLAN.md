# Enterprise action plan

**24 October 2026 | Operational instruction**

Answer four questions: **Are we exposed? Who owns the fix? What did we do? What evidence reasonably supports closure?**

## Minimum response

1. Identify the service or managed asset group and owner. Use existing inventories and configuration records; do not begin a complete cryptographic inventory.
2. Assign a [risk tier](RISK_TIERS.md) and record exposure as known or unknown. Identify whether the service uses an assured standard platform or a custom trust configuration.
3. Choose one of the closure routes below, apply the smallest effective remedy where needed, and record the outcome in the [six-field tracker](COMPLETION_TRACKER.md).
4. Escalate credible critical exposure, suspected exploitation, outages and unresolved blockers. Use existing incident/change processes.

## Three acceptable closure routes

| Route | When appropriate | Minimum evidence |
| --- | --- | --- |
| **A — Standard platform assurance** | The application uses a centrally maintained, remediated platform/store and has no relevant trust overrides. | Shared assurance reference, evidence that the deployment is covered, and owner confirmation that no relevant custom store/verifier or override is used. |
| **B — Application-specific validation** | Custom bundles, embedded runtimes, uncertain behaviour, or critical paths not adequately covered by Route A. | Applicable fix/configuration and activation evidence, representative validation using the relevant verifier, and normal integration checks. Link existing records. |
| **C — Not applicable to the assessed role** | Serve-only process with unaffected presented chain, or private-CA-only validation without affected public-trust fallback. | Short owner statement defining scope, supported by existing configuration/architecture evidence. Refer separately owned dependencies to their owners. |

Tier 1 requires strong assurance of coverage, but does not require duplicate testing where Route A demonstrably covers the critical path. Route C can close without an incident-specific change. Unknown exposure is not a valid Route C rationale.

## Shared assurance and proportionate testing

Security/PKI and platform owners should publish an assurance reference for each materially different standard configuration. Record the applicable vendor remedy, representative validation covering emSign and AAA routes, deployment coverage and known exclusions. Include normal service checks. A package compliance report alone does not establish shared assurance.

Applications reference that record rather than reproduce it. Existing deployment reports can establish that fleet members match the assessed configuration. No per-user screenshots, per-replica tests or per-application certificate hashes are required for an unchanged assured configuration.

Additional checks are triggered by a different runtime/verifier, custom store or mounted bundle, validation override, identity-dependent policy or materially different network validation path. Test the relevant difference, not every unrelated component. Security/PKI owns the approved test matrix; every team need not run every endpoint.

Do not require production internet access solely for the external checker. Use appropriate offline verification or a representative environment with documented equivalence. Product/version-specific vendor evidence can support closure where direct tests are impractical, provided applicability, deployment and limitations are recorded and Security accepts the residual uncertainty. Keep unexplained coverage gaps open.

## Responsibilities

| Team | Required contribution |
| --- | --- |
| **Security / PKI** | Coordinate tiers and exceptions; maintain affected-certificate references and approved tests; approve reusable assurance patterns and resolve ambiguous results. |
| **Desktop / browser** | Central compliance/deployment report plus representative checks per managed configuration. Track offline/unmanaged exceptions; no testing request to every user. |
| **Linux / Windows platform** | Assess and remediate standard stores/runtimes; validate each materially different configuration once and report deployment coverage. |
| **Application owners** | Confirm standard-platform applicability or identify custom trust behaviour. Validate exceptions through Route B; confirm normal application operation after changes. |
| **Container / DevSecOps** | Use existing image/package inventory; validate image digest plus trust configuration and map deployments to that evidence. Check changed mounts, sidecars and overrides; no duplicate tests of equivalent replicas. |
| **Network / proxy / WAF** | Cover the TLS legs actually verified by shared infrastructure. Distinguish proxy protection from the application's own trust. |
| **Supplier / service owners** | Obtain applicable product/version guidance and evidence where locally managed assurance is unavailable. |
| **SOC / incident response** | Scope investigation to credible exposure, available telemetry and suspicious activity. No mandatory historical investigation for every application. |

Assign one accountable owner per service or shared component. Application teams own confirmation of applicability; platform teams own their reusable evidence. Code-signing applicability is assessed centrally by endpoint/software-distribution owners, with application involvement only where relevant.

## Timing and reporting

- Assess credible critical exposure promptly and use the existing incident process for containment.
- Agree remediation dates with service/change owners based on tier, exposure and operational impact. Earlier 24/48-hour and 7/14/30-day figures are superseded; this pack sets no independently approved SLA.
- Record discovery/assignment dates for new findings. Do not reset existing overdue work when findings are reassigned.
- Request daily updates only for critical unresolved exposure, material blockers or outages. Other work follows the agreed change/reporting cadence.
- Continue normal automated security updates. Low-risk non-applicable cases need no special patch deadline.

## Remediation without a full application upgrade

A supported trust-data update, targeted package fix or configuration change may resolve the exposure while retaining the application version. Refer to [platform-specific options](REMEDIATION_OPTIONS.md). Rebuild/redeploy containers when required and ensure the change survives restart/scale-out. Do not restore vulnerable trust silently to resolve an outage.

## What is not universally required

No new scanner, CBOM, full certificate inventory, bespoke test harness, exhaustive source audit, per-instance certificate fingerprint or independent reviewer is required for every submission. Use [specialist validation](APPLICATION_VALIDATION.md) for Route B and central assurance work; use [CBOM guidance](CBOM_AND_CROSS_SIGNING.md) only where discovery would help or for longer-term improvement.

## Distribution brief

**To:** Named service or platform owner  
Please identify your scope and tier, select Route A, B or C, and return the six-field completion record with an existing evidence link. State any relevant custom trust configuration or uncertainty. Agree outstanding actions and dates through the existing incident/change process. Escalate critical exposure, outages and blockers promptly.

The brief is ready for internal routing; no recipients have been identified or messages sent.

## Basis

This is a proposed enterprise workflow informed by [Cloudflare's guidance](https://blog.cloudflare.com/emudhra-certificate-compromise/) and the [incident reference](https://emudhra.we-have-a-problem.com/). It does not assert that enterprise systems have been assessed or remediated.
