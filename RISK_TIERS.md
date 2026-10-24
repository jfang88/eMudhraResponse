# Risk tiers for the eMudhra response

**24 October 2026 | Proposed enterprise triage model, not a vendor severity rating**

## Management principle

Rank the affected **service or shared component** by its certificate-validation role. Use the highest unresolved exposure for management escalation, while keeping remediation and evidence scoped to the relevant component. A critical agent does not automatically impose Tier 1 testing on unrelated applications on the same host. Internet exposure alone does not determine priority. The decisive questions are whether the system accepts the affected trust chains, what an attacker could impersonate, and the business consequence.

A server presenting an unaffected certificate to visitors does not use its own public CA bundle to authenticate those visitors in ordinary one-way HTTPS. It can therefore have lower direct exposure to this incident than an internal server calling external APIs. However, an attacker could still impersonate that website to a vulnerable client, even if the real site's certificate comes from another CA. Server posture and client protection are separate.

For TLS impersonation, an attacker generally needs a way to intercept or redirect the victim's connection; possession of a CA key does not itself grant network access. For client-certificate authentication, an attacker can connect directly, but exploitation depends on the accepted certificate purpose, chain policy and identity/authorisation mapping.

## Ordered tiers

Unassessed systems have **unknown exposure**. Prioritise assessment using trust role and business impact; do not report unknown systems as confirmed vulnerable. The categories below guide sequencing, not proof requirements.

| Tier | Context and typical systems | Why this order / conditions | Response approach |
| --- | --- | --- | --- |
| **1 — Critical trust exposure** | High-impact systems validating certificates across an external or lower-trust boundary: payment/customer-data APIs, identity integrations, privileged automation, software-update/build pipelines and shared egress proxies. Internet-facing mTLS services where affected chains could satisfy the actual client-authentication policy. | Credentials, sensitive data, privileged actions or many downstream systems could be affected. These systems may be internet-facing OR internal servers with outbound access. Privileged desktops with exposed tooling can also qualify. | Assess promptly and contain credible critical exposure through the incident process; agree remediation dates with the service owner. |
| **2 — Broad internet client exposure** | General-purpose internet-accessing desktops/browsers, Electron apps, developer tools and ordinary outbound API clients. Internet-facing applications making non-critical external TLS calls. | Frequent external connections and many possible destinations. Browser protection must be verified; non-browser software on the same desktop remains separate. Promote to Tier 1 for high-impact credentials, build privileges or sensitive transactions. | Confirm protection using shared platform assurance where applicable; agree dates for unresolved exposure and continue automated updates. |
| **3 — Internal certificate-dependent systems** | Internal-only service-to-service connections that still use public CA trust or an unverified/mixed trust store; internal proxies and application-to-backend TLS. | Usually fewer external attack opportunities, but insiders, compromised hosts or routing/DNS manipulation can enable impersonation. Internal location is not proof of protection. Promote to Tier 1 or 2 where impact, lower-trust networks or attacker access warrants it. | Use existing change processes and an agreed date proportionate to exposure and impact. |
| **4 — Demonstrably limited direct exposure** | Serve-only websites with an unaffected presented chain, no relevant outbound certificate validation and no affected client-certificate trust. Internal systems demonstrably restricted to an unaffected private CA hierarchy, with no affected public-trust fallback. | Low direct exposure for the assessed role. Requires evidence; “we do not buy eMudhra certificates” is insufficient. Visitors or other consumers may still be vulnerable independently. | Close through Route C where non-applicability is supported; no incident-specific change required. Otherwise schedule applicable work normally. |

This pack sets no independently approved SLA. Record agreed dates and newly discovered findings without resetting existing overdue work. Evidence of exploitation or a critical outage triggers immediate incident response regardless of tier.

## Classify these common cases

| Scenario | Initial tier / treatment |
| --- | --- |
| Public static website; unaffected server certificate; no outbound application calls or mTLS | Tier 4 for the serving process after evidence review. Assess host agents, updates and deployment pipeline separately. |
| Public website calls an external payment or identity API | Tier 1 for those high-impact outbound paths, regardless of who issued the website's certificate. |
| Internal batch server sends customer data to a SaaS API | Tier 1; lack of inbound internet exposure does not reduce this outbound trust risk. |
| Internal low-impact service calls only managed internal endpoints but accepts public roots | Tier 3 initially; evaluate segmentation and consequences. |
| Internal service trusts only a dedicated unaffected private CA | Tier 4 after verifying no public-root fallback, alternate verifier or relevant external dependency. |
| Ordinary user browsing the internet | Tier 2 assessment; verified browser protections can close that path. A separate desktop agent or bundled client may remain unresolved. |
| Developer/admin workstation downloads software or uses privileged cloud credentials through an affected client | Tier 1 for that high-impact path; classify the remaining desktop paths separately. |
| Reverse proxy serves users and opens TLS to backends | Classify both legs. Its server certificate alone cannot determine the risk of backend certificate validation. |
| Container with no listening internet port but outbound external API calls | Tier 1 or 2 by impact. Image packaging does not determine tier. |

## Proportionate evidence for Tier 4

A short owner statement supported by existing configuration or architecture evidence is sufficient for Route C. State the assessed role: for example, serve-only with no relevant outbound/client-certificate validation, or private-CA-only with no public-trust fallback. Do not require a new exhaustive dependency map to prove a negative.

Use known dependencies and existing records to identify material exceptions. Refer shared host agents, update services, sidecars and deployment tooling to their accountable platform owners rather than duplicating their assessment. If a relevant dependency or fallback is uncertain, keep that part open and investigate proportionately; do not use uncertainty as evidence of non-applicability.

## Exposure tier and remediation status are different fields

Record tier and status separately in the six-field tracker; keep supporting context in the referenced existing record. A protected browser can remain in the Tier 2 asset class with status **Validated**; it does not require emergency remediation simply because it accesses the internet. Unknown is not Low. “Certificate found” is not proof of exploitability, and a clean package scan is not proof of protection.

Retain a separate **availability flag** for business certificates chaining to distrusted roots or services broken by root removal. A critical outage warrants urgent action even if confidentiality exposure is low. This incident's tiers do not replace general internet-facing vulnerability priorities.

## Basis and limitations

The tier model is an enterprise assessment derived from the authentication risks described by [Mozilla](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/) and [Cloudflare](https://blog.cloudflare.com/emudhra-certificate-compromise/). Neither source assigns these enterprise tiers. The [action plan](ACTION_PLAN.md) allows shared platform assurance, targeted application validation or supported non-applicability. Central assurance must address both emSign and AAA routes; equivalent applications can reference it without repeating tests. Use the [smallest effective supported remedy](ACTION_PLAN.md#remediation-without-a-full-application-upgrade); tiering does not require a full application upgrade.
