# Response completion tracker

**24 October 2026 | Six mandatory fields**

Use one record per service or managed asset group in the existing internal incident/change system. Identical deployments may share a record; distinguish material exceptions. Do not copy sensitive inventories or logs into a public repository.

| Mandatory field | What to provide |
| --- | --- |
| **1. Scope and owner** | Application/service or managed asset group, accountable owner and covered environment. |
| **2. Tier and rationale** | Tier 1–4 and a short explanation of trust role and business impact. Unknown exposure must be labelled unknown. |
| **3. Trust configuration** | Standard platform and assurance reference, or brief description of custom trust/uncertainty. |
| **4. Status and action** | Closure route A/B/C; status; remedy or reason no change is needed. |
| **5. Evidence reference** | Existing change/test/configuration record or shared platform assurance link; confirm deployment coverage and no relevant overrides when using Route A. |
| **6. Outstanding work** | Issue, owner and agreed target date; use “None” if closed. Record discovery/assignment date and any exception review date in the linked task. |

## Status choices

- **Assessing — exposure unknown**
- **Exposed — action required**
- **Remediating**
- **Closed — A: covered by platform assurance**
- **Closed — B: application-specific validation**
- **Closed — C: not applicable to assessed role**
- **Exception / blocked — owner and review date required**

A low tier is not proof of closure. Route C requires a scoped rationale. A clean scan or generic vendor statement alone is insufficient; evidence must apply to the deployed configuration.

## Example of shared evidence

A platform team validates a managed runtime and trust configuration and publishes assurance record PLATFORM-123. An application owner references PLATFORM-123, links the deployment record showing the applicable configuration and confirms there is no relevant custom trust override. The application need not repeat the central certificate tests or copy certificate hashes into this tracker.

## Conditional technical evidence

For custom changes or uncertain behaviour, retain relevant store hashes, verifier details, test results and coverage limitations in the existing technical record. Use the [validation appendix](APPLICATION_VALIDATION.md) to select checks. These details are not additional mandatory columns. Security review is needed for disputed assurance or risk exceptions, not a second signature on every standard submission.

## Coordination and reporting

The response lead maps functional teams in [ACTION_PLAN.md](ACTION_PLAN.md) to actual internal owners and queues; no distribution has yet occurred.

Report counts by tier and status using a defined unit (service, asset group or deployment); avoid mixing units or double-counting shared platforms. Highlight critical unresolved exposure, outages, blockers and decisions needed. Use existing deployment dashboards for fleet coverage.

Daily reporting applies only to critical unresolved exposure, material blockers or outages. Other records follow agreed change cadence. Preserve existing due dates when reassigning work.
