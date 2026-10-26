# Response completion tracker

**Version 2 · 26 October 2026 · Six mandatory fields**

Use one record per service or managed asset group in the existing internal
incident or change system. Identical deployments may share a record;
distinguish material exceptions. The workbook (`eMudhra-response-workbook.xlsx`,
*Service tracker* sheet) has these columns, with dropdowns and the priority
calculated automatically. Keep populated copies in the internal system. Don't
copy sensitive inventories, hostnames or logs into a public repository.

## Mandatory fields

| Field | What to provide |
|---|---|
| **1. Scope and owner** | Application, service or managed asset group; accountable owner; covered environment |
| **2. Priority and rationale** | Impact (H/M/L), verifier susceptibility (A, B, C, D or Unknown), resulting priority P1–P4 from the [matrix](RISK_TIERS.md#4-priority-matrix), and a sentence on trust role and business impact. Label unknown exposure as Unknown |
| **3. Trust configuration** | Standard platform and its assurance reference, or a brief description of the custom trust or the uncertainty |
| **4. Status and action** | Closure route A, B or C; status (below); the remedy, or the reason no change is needed |
| **5. Evidence reference** | Existing change, test or configuration record, or the shared platform assurance link. For Route A, confirm deployment coverage and no relevant overrides |
| **6. Outstanding work** | Issue, owner and agreed target date, or "None" if closed. Record the discovery or assignment date and any exception review date |

**Recommended columns** (in the workbook, not mandatory): playbook used
(PB-00 to PB-16), availability flag (Y/N), code-signing relevance (Y/N),
version 1 tier for records created before 26 October.

## Status choices

- **Assessing — exposure unknown**
- **Exposed — action required**
- **Remediating**
- **Closed — A: covered by platform assurance**
- **Closed — B: application-specific validation**
- **Closed — C: not applicable to assessed role**
- **Exception / blocked — owner and review date required**

A low priority is not proof of closure. Route C requires a scoped rationale. A
clean scan or generic vendor statement alone is not enough; evidence must
apply to the deployed configuration. Exceptions need a compensating control,
an approver and an expiry date (workbook *Exceptions* sheet; policy in the
[action plan](ACTION_PLAN.md#6-phase-2-the-long-tail)).

## Example of shared evidence

A platform team validates a managed runtime and trust configuration and
publishes assurance record PLATFORM-123. An application owner references
PLATFORM-123, links the deployment record showing the applicable
configuration, and confirms there is no relevant custom trust override. The
application doesn't repeat the central certificate tests or copy certificate
hashes into the tracker.

## Conditional technical evidence

For custom changes or uncertain behaviour, keep the relevant store hashes,
verifier details, test results and coverage limitations in the existing
technical record. Use [PB-00](playbooks/PB-00-test-and-evidence.md) and the
[validation appendix](APPLICATION_VALIDATION.md) to choose checks. These
details are not extra mandatory columns. Security review is needed for
disputed assurance or risk exceptions, not as a second signature on every
standard submission.

## Reporting

The incident lead maps the functional teams in the
[action plan](ACTION_PLAN.md#9-responsibilities-and-first-actions) to named
owners and queues. No distribution has happened yet.

Count records by **priority and status**, using the service as the unit.
Don't mix units or double-count shared platforms. The workbook *Dashboard*
calculates these headline indicators:

| Indicator | Why it matters |
|---|---|
| P1 records not closed | The main measure of residual risk |
| Records with susceptibility Unknown | Assessment backlog; unknown is not low |
| Records overdue against their target date | Delivery against the indicative windows |
| Availability-flagged records not closed | The 27 October and root-removal impact |
| Browser and desktop configurations meeting the proviso | Phase 1 coverage |
| Long-tail items with vendor status Pending or No response | Vendor dependency |
| Approved exceptions expiring within 14 days | Exceptions coming due for review |
| Decisions still open | Governance backlog |
| Hunts completed (of H1–H6) | Retrospective assurance |

Daily reporting applies only to open P1 items, material blockers and outages.
Other records follow the agreed change cadence. Keep existing due dates when
work is reassigned. The incident closes against the criteria in the
[action plan](ACTION_PLAN.md#2-governance).

## References

**Related documents**

- [Action plan](ACTION_PLAN.md): closure routes, exceptions, reporting cadence, closure criteria
- [Prioritisation](RISK_TIERS.md): impact, susceptibility and the priority matrix
- [Playbooks](playbooks/README.md): steps and evidence for each team; [PB-00](playbooks/PB-00-test-and-evidence.md) for testing and evidence
- [Application validation](APPLICATION_VALIDATION.md): selecting checks for Route B
- Workbook: `eMudhra-response-workbook.xlsx` (*Service tracker*, *Exceptions*, *Dashboard*)

This tracker contains no external sources.
