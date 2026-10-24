# Risk tiers for the eMudhra response

**24 October 2026 | Proposed enterprise triage model, not a vendor severity rating**

## Management principle

Rank **certificate-validation paths**, then assign each asset the highest applicable tier. Internet exposure alone does not determine priority. The decisive questions are whether the system accepts the affected trust chains, what an attacker could impersonate, and the business consequence.

A server presenting an unaffected certificate to visitors does not use its own public CA bundle to authenticate those visitors in ordinary one-way HTTPS. It can therefore have lower direct exposure to this incident than an internal server calling external APIs. However, an attacker could still impersonate that website to a vulnerable client, even if the real site's certificate comes from another CA. Server posture and client protection are separate.

For TLS impersonation, an attacker generally needs a way to intercept or redirect the victim's connection; possession of a CA key does not itself grant network access. For client-certificate authentication, an attacker can connect directly, but exploitation depends on the accepted certificate purpose, chain policy and identity/authorisation mapping.

## Ordered tiers

Assume the affected chain is accepted or protection is unverified unless the row specifies otherwise.

| Tier | Context and typical systems | Why this order / conditions | Proposed response |
| --- | --- | --- | --- |
| **1 — Critical trust exposure** | High-impact systems validating certificates across an external or lower-trust boundary: payment/customer-data APIs, identity integrations, privileged automation, software-update/build pipelines and shared egress proxies. Internet-facing mTLS services where affected chains could satisfy the actual client-authentication policy. | Credentials, sensitive data, privileged actions or many downstream systems could be affected. These systems may be internet-facing OR internal servers with outbound access. Privileged desktops with exposed tooling can also qualify. | Assess immediately; target containment within 24 hours and validated remediation within 48 hours of T0. |
| **2 — Broad internet client exposure** | General-purpose internet-accessing desktops/browsers, Electron apps, developer tools and ordinary outbound API clients. Internet-facing applications making non-critical external TLS calls. | Frequent external connections and many possible destinations. Browser protection must be verified; non-browser software on the same desktop remains separate. Promote to Tier 1 for high-impact credentials, build privileges or sensitive transactions. | Confirm protection promptly; target validation/remediation within seven days. Continue automated updates immediately. |
| **3 — Internal certificate-dependent systems** | Internal-only service-to-service connections that still use public CA trust or an unverified/mixed trust store; internal proxies and application-to-backend TLS. | Usually fewer external attack opportunities, but insiders, compromised hosts or routing/DNS manipulation can enable impersonation. Internal location is not proof of protection. Promote to Tier 1 or 2 where impact, lower-trust networks or attacker access warrants it. | Establish scope within seven days; target remediation/validation within 14 days. |
| **4 — Demonstrably limited direct exposure** | Serve-only websites with an unaffected presented chain, no relevant outbound certificate validation and no affected client-certificate trust. Internal systems demonstrably restricted to an unaffected private CA hierarchy, with no affected public-trust fallback. | Low direct exposure for the assessed role. Requires evidence; “we do not buy eMudhra certificates” is insufficient. Visitors or other consumers may still be vulnerable independently. | Validate the low-exposure rationale within 14 days; complete applicable trust hygiene in the normal maintenance window, proposed within 30 days. |

Targets are planning recommendations, not mandatory vendor deadlines. The response lead may shorten them based on operational impact. Evidence of exploitation triggers immediate incident response regardless of tier.

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

## Evidence required for Tier 4

The owner must document the actual TLS roles, presented chain, outbound dependency map and effective trust policy. Include authentication/JWKS retrieval, telemetry, webhooks, package/update agents, scheduled jobs, sidecars and deployment tooling where applicable. Separate independent processes rather than assuming the entire host is serve-only.

For private-CA-only claims, verify that the relevant client actually replaces public trust and has no fallback. For serve-only claims, check whether the server accepts client certificates or makes backend TLS calls. If a dependency is uncertain, keep its exposure **Unknown** and assign the tier appropriate to the plausible use case pending prompt assessment.

## Exposure tier and remediation status are different fields

Record tier, business impact, trust role, reachability, evidence and status separately. A protected browser can remain in the Tier 2 asset class with status **Validated**; it does not require emergency remediation simply because it accesses the internet. Unknown is not Low. “Certificate found” is not proof of exploitability, and a clean package scan is not proof of protection.

Retain a separate **availability flag** for business certificates chaining to distrusted roots or services broken by root removal. A critical outage warrants urgent action even if confidentiality exposure is low. This incident's tiers do not replace general internet-facing vulnerability priorities.

## Basis and limitations

The tier model is an enterprise assessment derived from the authentication risks described by [Mozilla](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/) and [Cloudflare](https://blog.cloudflare.com/emudhra-certificate-compromise/). Neither source assigns these enterprise tiers. Validate effective distrust of both emSign and AAA routes using the [application runbook](APPLICATION_VALIDATION.md). Use the [smallest effective supported remedy](ACTION_PLAN.md#remediation-without-a-full-application-upgrade); tiering does not require a full application upgrade.
