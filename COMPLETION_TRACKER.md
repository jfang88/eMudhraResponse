# Response completion tracker

**Initial state: not assessed.** No enterprise validation or team distribution has been performed by creating this repository.

## Team routing register

Replace role placeholders with actual owners and internal task links. Do not publish sensitive asset inventories, credentials or diagnostic logs in a public repository.

| Work package | Named owner / deputy | Task / distribution link | Acknowledged | Due | Status |
| --- | --- | --- | --- | --- | --- |
| SEC | TBD | Pending | No | T0 + 24h | Not started |
| EUC | TBD | Pending | No | Set by risk tier | Not started |
| LNX | TBD | Pending | No | Set by risk tier | Not started |
| WIN | TBD | Pending | No | Set by risk tier | Not started |
| APP | TBD | Pending | No | Set by risk tier | Not started |
| CTR | TBD | Pending | No | Set by risk tier | Not started |
| NET | TBD | Pending | No | Set by risk tier | Not started |
| VND | TBD | Pending | No | Set by risk tier | Not started |
| SOC | TBD | Pending | No | Set by risk tier | Not started |

## Per-application / verification-path record

Copy this record into the internal incident/task system for each distinct path.

| Field | Required value |
| --- | --- |
| Asset/application and business criticality | |
| Exposure tier (1–4), rationale, assessor and review date | |
| TLS role: presents server cert / verifies server cert / verifies client cert / code signing | |
| Internet/partner/internal reachability and outbound dependencies | |
| Impact: credentials, data, privilege and downstream reach | |
| Tier 4 evidence or promotion triggers, where applicable | |
| Availability impact flag and separate urgency | |
| Tier-based deadline | |
| Team, owner, environment and scope denominator | |
| Runtime/library/verifier and service identity | |
| Effective store location, provenance and file hash (if applicable) | |
| SBOM / CBOM links, collector versions and exclusions | |
| Incident reference retrieval date/hash | |
| Certificate SHA-256 / SPKI matches and observed roles | |
| Effective trust purpose and distrust mechanism | |
| emSign route test / AAA route test (separate results) | |
| Code-signing applicability, owner and validation reference | |
| Unsupported stores / embedded-root coverage gaps | |
| Proxy / TLS termination route | |
| Affected status and supporting evidence | Unknown until assessed |
| Vendor advisory / fix applied | |
| Remediation method: trust-data / targeted package / configuration / runtime | |
| Retained application version and vendor support applicability | |
| Trust-store/blocklist before-and-after hashes | |
| Persistence/drift control and owner | |
| Temporary remedy review/expiry and residual patch requirement | |
| Deployed version/image digest; restart or rollout evidence | |
| Negative tests: chains tested, exact errors and timestamps | |
| Positive integration test | |
| Coverage: tested replicas/configurations and equivalence basis | |
| Evidence links (internal) and independent reviewer | |
| Exception/control, risk owner, expiry and next test | |
| Status | Not assessed / Exposed / Remediating / Inconclusive / Validated / Exception |
| Closure approval and date | |

## Management report template

Report the following separately for each tier, with an additional availability-impact summary. Unknown exposure is not a low-risk classification.

As of [timestamp]: [N/total] assets assessed; [N/affected] remediated; [N/affected] validated. [N] remain unassessed, [N] exposed, [N] inconclusive and [N] under time-limited exceptions. Critical unresolved services: [internal reference]. Service impacts: [summary]. Decisions needed: [owner, action, deadline].

Do not combine patch compliance and validation into one percentage. Retain the inventory date and coverage denominator so newly discovered assets are visible.
