# eMudhra enterprise response

**Senior IT management and service owners · Version 2 · 26 October 2026 · Proposed: decisions pending ([action plan, Appendix A](ACTION_PLAN.md#appendix-a-decision-register))**

## Executive summary

Eight eMudhra intermediate CA private keys were generated weakly in April 2025
and can be recovered by anyone from public certificate data; one has been
published. Six of the intermediates issue TLS certificates and two sign code.
With a key, an attacker can forge certificates that unprotected clients
accept. Anyone able to intercept or redirect a connection can then
impersonate a service and read or change its traffic. Where a service
accepts client certificates, the attacker can connect directly. **Exposure
depends on what a system trusts, not on whether we buy eMudhra
certificates.** [1][2]

**Where we stand.** Browser and operating-system vendors blocked the keys
between 17 and 20 October without user action. We treat browsers and OS trust
stores as protected only under the
[browser and desktop proviso](ACTION_PLAN.md#5-phase-1-browser-and-desktop-proviso):
four enterprise conditions, confirmed by a coverage report. The remaining
risk sits in the **long tail**: software that keeps its own trust list.
That includes Java, Python, Node, Go and Rust applications, containers,
CI/CD, appliances, agents, OT/IoT, in-house mobile apps and suppliers.

The affected keys also have Sectigo AAA cross-certificates. Where AAA is
still trusted (Windows, Java, older or vendor bundles), assurance must cover
both routes. This is not a compromise of Sectigo's root key. [3]

**What we ask.**

1. Every service returns one [six-field record](COMPLETION_TRACKER.md),
   worked in [priority order](RISK_TIERS.md) (impact × verifier
   susceptibility).
2. Fix the chokepoints first: TLS-inspection proxies, identity integrations,
   build pipelines and artifact proxies.
3. Replace any of our certificates under **emSign ECC Root CA - G3** now:
   Chrome stops trusting that root on **Tuesday 27 October 2026**. [4]
4. Run the targeted retrospective hunt. Any chain to these keys is
   presumptively forged.

**Management approach:** a proportionate response using shared platform
assurance, targeted exception testing and existing change processes. A
trust-data update or targeted fix may avoid a full application upgrade. Our
enterprise's exposure and compromise status have not yet been assessed.

## Situation as at 26 October 2026

| Date (2026) | Event | Confidence |
|---|---|---|
| 30 Apr 2025 | Eight keys generated with P-384 private values below 256, recoverable from public data [5][6] | Secondary (citing Mozilla) |
| 17 Oct | Report received. Chrome blocks the keys by CRLSet and Firefox by OneCRL, with no browser update or restart needed [4][7] | Primary (Google) |
| 19 Oct | Apple blocks. Android receives protection through Google Play services. Sectigo revokes the eight AAA cross-certificates. Go 1.27.3 and 1.26.10 published (programs must be rebuilt) [3][5][8] | Secondary; Go release verified |
| 19–23 Oct | Rust webpki-roots 1.0.10 and Python certifi 2026.10.18 and later drop emSign ECC Root CA - G3 [9][10] | Verified (package contents) |
| 20 Oct | Microsoft blocks. Ubuntu 24.04 `ca-certificates` blocklists emSign ECC Root CA - G3 [6][11] | Secondary; Ubuntu verified |
| 20–21 Oct | Node.js, Bun, Debian and FreeBSD fixes pending. macOS's built-in `curl` still accepted the affected chain after Apple's block [12][13] | Secondary |
| 21 Oct | No confirmed malicious use. Google found no TLS certificate under the six intermediates in CT logs before the block. That protects CT-enforcing browsers but says nothing about clients that don't check CT [5][14] | Secondary |
| 26 Oct | Node.js 22.22.0's built-in roots still include emSign ECC Root CA - G3 | Verified |
| **27 Oct** | **Chrome removes emSign ECC Root CA - G3** from its root store. Legitimate certificates under it fail in Chrome; eMudhra estimates about 30,000 need migration [4] | Primary (Google) |

The incident lead owns this table and rechecks it daily while the incident is
open (workbook *Facts* sheet). "Verified" items were checked directly against
package contents on 26 October. "Secondary" items come from press coverage of
vendor statements, because the primary pages could not be fetched when this
version was prepared. Confirm them against the vendor before acting on a
detail.

## What each audience receives

| Audience | Send them | Purpose |
|---|---|---|
| **Senior IT management** | This README, `eMudhra-response-briefing.pptx` (or `.pdf`), [Action plan Appendix A](ACTION_PLAN.md#appendix-a-decision-register) | Situation, priorities, plan and the decisions requested |
| **Incident lead, Security/PKI** | [Action plan](ACTION_PLAN.md), [Prioritisation](RISK_TIERS.md), [Completion tracker](COMPLETION_TRACKER.md), `eMudhra-response-workbook.xlsx` | Running the response, decisions and reporting |
| **Application owners and developers** | [**Team guide: `playbooks/README.md`**](playbooks/README.md) | What to do, what to return and by when; links to their runtime playbook ([Java](playbooks/PB-04-java.md), [Python](playbooks/PB-05-python.md), [Node/Electron](playbooks/PB-06-nodejs-and-electron.md), [Go/Rust/PHP/Ruby](playbooks/PB-07-go-rust-php-ruby.md), [.NET](playbooks/PB-02-windows-and-dotnet.md)), [testing](playbooks/PB-00-test-and-evidence.md) and [identity/mTLS](playbooks/PB-16-identity-and-mtls.md) |
| **Platform, network, container and desktop teams** | The same [team guide](playbooks/README.md), then PB-01 to PB-03 and PB-08 to PB-10; [Remediation options](REMEDIATION_OPTIONS.md) for depth | Shared fixes and Route A assurance records |
| **SOC** | [PB-14](playbooks/PB-14-soc-threat-hunt.md), [PB-12](playbooks/PB-12-code-signing-and-application-control.md) | Retrospective hunt and monitoring |
| **Service desk** | [PB-15](playbooks/PB-15-service-desk.md) | Knowledge article for user contacts from 27 October |
| **Supplier and service owners** | [PB-11](playbooks/PB-11-vendor-products-appliances-and-suppliers.md) | Vendor and supplier requests; long-tail tracking |

## Document map

| Document | Purpose |
|---|---|
| [Action plan](ACTION_PLAN.md) | How the response runs: governance, phases, availability and hunt, responsibilities, the 30-day plan, the decision register |
| [Prioritisation](RISK_TIERS.md) | Impact × verifier susceptibility, the priority matrix, fix-first paths, worked cases |
| [Completion tracker](COMPLETION_TRACKER.md) | The six-field record, status values and reporting indicators |
| [Playbooks](playbooks/README.md) | Team guide and 17 playbooks, each with *applies if / fix / verify / record / escalate* |
| [Remediation options](REMEDIATION_OPTIONS.md) | Specialist reference: verified fixes and version-aware recipes that avoid full upgrades |
| [Application validation](APPLICATION_VALIDATION.md) | Specialist reference: discovery, testing and acceptance for Route B and central assurance |
| [CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md) | Optional deeper discovery, and the AAA cross-sign route |
| `eMudhra-response-workbook.xlsx` | Decision register, tracker with calculated priority, assurance, proviso, long tail, availability, exceptions, hunt log, facts, dashboard |
| `eMudhra-response-briefing.pptx` / `.pdf` | 16-slide briefing for senior IT management and service owners |

## Document history

| Version | Date | Change |
|---|---|---|
| 1 | 24 Oct 2026 | Initial pack: action plan, four risk tiers, completion tracker, remediation options, validation appendix, CBOM and cross-signing reference |
| 2 | 26 Oct 2026 | Enterprise review incorporated. **Prioritisation** now combines impact with verifier susceptibility, so vendor-protected browsers move down and own-bundle clients move up. **New:** browser and desktop proviso; long-tail discovery through egress telemetry, with a cut-off and exception policy; availability track for Chrome's 27 October root removal; targeted retrospective hunt; code signing and identity paths treated as P1; governance, indicative target windows, a 30-day plan and a decision register; mobile, supplier, service-desk and legal coverage; dated facts with confidence labels. **Added** 17 playbooks, the workbook and the briefing deck. Earlier statements that the pack sets no target windows are replaced by indicative windows pending decision D-03 |

## References

1. Cloudflare: [8 eMudhra certificates are compromised. Here's what you need to know](https://blog.cloudflare.com/emudhra-certificate-compromise/)
2. Incident reference: [Impact, maintained package list and test fixtures](https://emudhra.we-have-a-problem.com/)
3. Sectigo: [Advisory on compromised eMudhra intermediate certificates](https://www.sectigo.com/resource-library/advisory-on-compromised-emudhra-intermediate-certificates)
4. Google Security Blog: [Protecting Chrome and Android users from compromised eMudhra intermediate CA certificates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
5. BleepingComputer: [Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
6. Free Press Journal: [Why did Apple, Google, Microsoft and Mozilla block certificates from eMudhra?](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra)
7. Mozilla: [Protecting Firefox users following the compromise of eMudhra intermediate certificates](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/)
8. Technobezz: [Google blocks compromised eMudhra keys in Chrome and Android](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android); Go: [release history](https://go.dev/doc/devel/release)
9. crates.io: [webpki-roots versions](https://crates.io/crates/webpki-roots/versions)
10. PyPI: [certifi release history](https://pypi.org/project/certifi/#history)
11. Launchpad: [Ubuntu ca-certificates bug 2171507](https://bugs.launchpad.net/ubuntu/+source/ca-certificates/+bug/2171507)
12. Rescana: [eMudhra intermediate CA key compromise, October 2026](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026)
13. lilting.ch: [Recovering eMudhra's weak P-384 CA keys and testing curl, Node.js and uv on macOS](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
14. iTnews: [Cloudflare warns fixing botched eMudhra certs goes beyond browsers](https://www.itnews.com.au/news/cloudflare-warns-fixing-botched-emudhra-certs-goes-beyond-browsers-629804)

Further coverage used in preparing version 2:
[CSO Online](https://www.csoonline.com/article/4238338/google-mozilla-revoke-emudhra-intermediates-after-private-key-compromise.html),
[The Hacker News](https://thehackernews.com/2026/10/eight-emudhra-ca-keys-compromised.html),
[Endor Labs](https://www.endorlabs.com/learn/emudhra-intermediate-ca-compromise-the-trust-stores-you-patched-and-the-ones-you-didnt-know-you-had),
[oss-security](https://www.openwall.com/lists/oss-security/2026/10/20/30),
[Veeam KB4952](https://www.veeam.com/kb4952).

This repository contains proposed procedures, not completed enterprise
assessments. No team messages have been sent. Recheck vendor guidance before
implementation.
