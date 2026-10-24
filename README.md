# eMudhra enterprise response

**Senior IT management and service owners — 24 October 2026**

## Executive summary

Eight eMudhra intermediate CA private keys were compromised. Vulnerable clients could accept forged certificates, allowing an attacker able to intercept or redirect connections to impersonate services and expose or modify traffic. Exposure depends on what a system trusts, not whether the enterprise buys eMudhra certificates. [1][2]

Prioritise high-impact certificate-validation paths. An internal server calling a sensitive external API may warrant more urgent action than a public website that only presents its own unaffected certificate. Already-protected browsers need confirmation of coverage, not unnecessary changes.

Affected keys also have Sectigo AAA cross-certificates: assurance must address both trust routes. This is not evidence of compromise of Sectigo's root key. [3]

**Management approach:** coordinate a proportionate response using shared platform assurance, targeted exception testing and existing change processes. A trust-data update or targeted fix may avoid a full application upgrade. Our enterprise's exposure and compromise status have not been assessed by this pack.

## What teams need to return

Use the [six-field tracker](COMPLETION_TRACKER.md): scope/owner, tier/rationale, trust configuration, status/action, evidence link and outstanding work.

Choose one closure route:

| Route | Evidence approach |
| --- | --- |
| **A — Standard platform assurance** | Reference centrally validated remediation and deployment coverage; confirm no relevant custom overrides. |
| **B — Application-specific validation** | Target checks at custom, uncertain or critical paths not covered by shared assurance. |
| **C — Not applicable** | Short scoped rationale supported by existing configuration or architecture records. |

No universal CBOM, new scanner, bespoke harness or per-instance certificate evidence is required. Tier 1 needs stronger assurance, but equivalent configurations can reuse evidence.

## Reading order

| Document | Audience / purpose |
| --- | --- |
| [Action plan](ACTION_PLAN.md) | Required operational workflow, owners and closure routes |
| [Risk tiers](RISK_TIERS.md) | Prioritisation by trust role and business impact |
| [Completion tracker](COMPLETION_TRACKER.md) | Minimum submission using existing evidence |
| [Remediation options](REMEDIATION_OPTIONS.md) | Specialist reference: targeted fixes without full upgrades |
| [Validation appendix](APPLICATION_VALIDATION.md) | Selective checks for central assurance and application exceptions |
| [CBOM and cross-signing reference](CBOM_AND_CROSS_SIGNING.md) | Optional discovery support and longer-term improvement |

Tier 1 covers critical trust exposure; Tier 2 broad internet client exposure; Tier 3 internal public/mixed trust; Tier 4 demonstrably limited exposure. Unknown exposure is not confirmed vulnerability or an automatic low-risk classification. Agree dates through the existing incident/change process; suspected exploitation or critical outages require immediate escalation.

## Sources and status

1. [Cloudflare impact and remediation](https://blog.cloudflare.com/emudhra-certificate-compromise/).
2. [Incident information and maintained package list](https://emudhra.we-have-a-problem.com/).
3. [Sectigo advisory](https://www.sectigo.com/resource-library/advisory-on-compromised-emudhra-intermediate-certificates).

Technical sources were checked on 24 October 2026; recheck applicable vendor guidance before implementation. This repository contains proposed procedures, not completed enterprise assessments. No team messages have been sent.
