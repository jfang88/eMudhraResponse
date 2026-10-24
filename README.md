# eMudhra enterprise response

**Prepared for senior IT management — 24 October 2026**

## Executive summary

Eight eMudhra intermediate CA private keys were compromised, enabling forged certificates that vulnerable clients could accept as legitimate. An attacker able to intercept or redirect connections could impersonate services and expose or modify sensitive traffic. This is a certificate-authentication failure; it does not mean all TLS encryption has been broken. [1][2]

**Enterprises may be exposed even if they do not buy eMudhra certificates.** The issue is whether software trusts an affected chain. Vendor browser protections do not establish that servers, applications or containers are protected. [1][2]

Potential business consequences include credential or data exposure, transaction manipulation and operational disruption when distrusted certificates are rejected. Cloudflare reported no identified active exploitation on its services; this is not evidence that other organisations were unaffected. Our organisation's exposure and compromise status have not been assessed by this document. [1]

**Recommended management decision:** activate an urgent, coordinated response led by Security, with named owners across desktop, server, application, container and supplier teams. Prioritise high-impact certificate-validation paths using the [risk tiers](RISK_TIERS.md), regardless of whether the host is internet-facing. Require deployment evidence and application-level validation before closure, and explicitly track unsupported or untested systems.

**Cross-signing matters:** the affected keys also have Sectigo AAA Certificate Services cross-certificates. Removing only the eMudhra route may leave an accepted AAA route. This is not evidence that Sectigo's root key was compromised. Require checks of both routes and distinguish package scanning, certificate inventory and actual runtime trust. See the [tooling and cross-signing guide](CBOM_AND_CROSS_SIGNING.md) for sources and assessment criteria.

**Application upgrades are not always required.** The plan now permits vendor trust-data/blocklist updates, targeted CA-package fixes and validated configuration changes while retaining the application version. Some embedded verifiers still require a patch or rebuild. See [remediation without a full upgrade](ACTION_PLAN.md#remediation-without-a-full-application-upgrade).

## Risk order

| Tier | Main context |
| --- | --- |
| 1 — Critical | High-impact external certificate validation, privileged clients and applicable exposed mTLS authentication |
| 2 — Broad internet exposure | General internet-accessing desktops, browsers and ordinary external API clients |
| 3 — Internal trust exposure | Internal service-to-service validation using public or unverified mixed trust |
| 4 — Limited direct exposure | Evidenced serve-only systems with unaffected chains, or private-CA-only paths without public fallback |

A public server that only presents its own unaffected certificate can be lower risk for this incident than an internal server trusting an external API's certificate. **Not purchasing eMudhra certificates does not protect a client that still trusts the affected chains.** Visitors to a serve-only website still need protected clients. Tier and remediation status are separate; verified browser protection can close that browser path while other desktop software remains outstanding.

See [classification examples and downgrade evidence](RISK_TIERS.md). These are proposed enterprise tiers, not vendor ratings.

## Response pack

| Document | Audience / purpose |
| --- | --- |
| [Risk tiers](RISK_TIERS.md) | Ordered exposure categories, examples and evidence requirements |
| [Team action plan](ACTION_PLAN.md) | Allocate work, proposed deadlines, distribution and management reporting |
| [Application validation runbook](APPLICATION_VALIDATION.md) | Discover hidden trust stores and prove deployed applications reject affected chains |
| [CBOM and cross-signing guide](CBOM_AND_CROSS_SIGNING.md) | Tool capabilities, inventory pipeline and Sectigo/AAA scope |
| [Completion tracker](COMPLETION_TRACKER.md) | Record ownership, scope, evidence and exceptions |

These are proposed response procedures, not a claim that enterprise systems have been scanned, patched or tested. Team names are functional roles; the response lead must map them to actual owners. No messages have been sent to teams.

## Sources

1. [Cloudflare: compromise, impact and remediation](https://blog.cloudflare.com/emudhra-certificate-compromise/).
2. [Incident information, test endpoints and maintained package list](https://emudhra.we-have-a-problem.com/).

Sources checked on 24 October 2026. Use the maintained list to discover relevant fixes, then verify the applicable release with the software vendor; package versions and coverage can change.
