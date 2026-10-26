---
title: "Review of the eMudhra enterprise response pack"
subject: "jfang88/eMudhraResponse at commit 18621f0 (pack dated 24 October 2026)"
version: "0.1.0"
status: "Draft for the response lead to disposition"
date: "2026-10-26"
reviewer: "Claude (agent review, at owner's request)"
classification: "Internal"
artefacts:
  - "REVIEW.md (this document)"
  - "playbooks/ (17 per-stack and per-team remediation playbooks)"
  - "eMudhra-response-workbook.xlsx (decisions, tracker, assurance, long tail, exceptions, dashboard)"
  - "eMudhra-response-briefing.pptx and .pdf (16-slide briefing for senior IT management and service owners)"
---

# Review of the eMudhra enterprise response pack

> **How to read this.** §1 gives the bottom line. §2 lists facts that changed or
> were missing on 24 October. §3 answers "does the risk tiering make sense?".
> §4 proposes the browser and desktop proviso. §5 deals with the long tail of
> internet-facing, application and certificate-dependent systems. §6 lists the
> remaining enterprise gaps as findings with a severity rating. §7 explains how
> the playbooks make the technical remediation findable. §8 lists the decisions
> captured in the workbook.

**Fact-checking caveat.** The review environment's network policy blocked the
primary pages (Cloudflare, Mozilla, Sectigo, the incident site, the vendor
advisories). The facts in §2 come from search-indexed copies of those pages and
from press coverage dated 20–23 October 2026. Each fact carries a confidence
label. The response lead should confirm the **Secondary** items against the
primary source before the pack is reissued.

---

## 1. Bottom line

1. **The pack's core idea is right. Keep it.** Exposure depends on what a
   system *trusts*, not on whether we buy eMudhra certificates, so priority
   follows certificate-validation role rather than internet exposure. Closure
   routes A/B/C, shared platform assurance and "smallest effective remedy" are
   the right ways to scale the response across an enterprise.
2. **The tiering is sound in principle but sorts by the wrong thing.** It ranks
   by asset class and impact, while exploitability depends almost entirely on
   the *verifier*: browsers and OS stores were protected by vendors on
   17–20 October, but bundled, compiled and embedded trust stores were not. A
   second axis (verifier susceptibility) moves browsers out of the queue and
   pulls non-browser clients up. See §3.
3. **Yes, add a proviso on desktops and browsers.** "Fixed" should mean "vendor
   protection delivered *and* enterprise coverage confirmed". It does not cover
   other software on the same desktop or traffic through a TLS-inspection
   proxy. See §4.
4. **The long tail is the programme.** Once browsers and OS stores are
   confirmed, all remaining risk sits in non-browser verifiers: server-side API
   clients, runtimes with their own bundles, containers, CI/CD, appliances,
   agents, OT/IoT, in-house mobile apps and suppliers. The fastest way to list
   them without a CBOM is **egress telemetry**. See §5.
5. **Three time-critical items are missing.** (a) Chrome removes emSign ECC
   Root CA - G3 on **Tuesday 27 October 2026**, an *availability* event for any
   legitimate certificate chaining to that root. (b) The keys were weak from
   generation on 30 April 2025, so clients were exposed for up to roughly 18
   months; a targeted **retrospective hunt**, particularly for code-signing, is
   cheap and is not in the pack. (c) **Code signing** has no tier, owner date
   or hunt. See §6.
6. **App, IT and support teams will struggle to find their steps.** The
   technical content is good but sits in a 26 KB document ordered by vendor
   mechanism, with caveats repeated throughout. The 17 playbooks in
   [`playbooks/`](playbooks/README.md) reorganise it by "who you are" and "what
   you run", each with *applies if / fix / verify / record / escalate*.

---

## 2. Facts the pack should carry (as at 26 October 2026)

The pack does not mention any vendor action or date. Without them, readers
cannot judge what "already-protected browsers" means, or what is still open.

| Date (2026 unless stated) | Event | Confidence | Source |
|---|---|---|---|
| 30 Apr 2025 | Eight intermediate CA keys generated in a key ceremony with P-384 private values below 256, recoverable by anyone from public certificate data. | Secondary (citing Mozilla's incident report) | [Free Press Journal](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra); [BleepingComputer](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates) |
| — | Six intermediates issue TLS certificates; two are code-signing intermediates, reported as trusted by Microsoft. eMudhra states one intermediate was deployed in production and none issued certificates to end entities. | Secondary | [CSO Online](https://www.csoonline.com/article/4238338/google-mozilla-revoke-emudhra-intermediates-after-private-key-compromise.html); [Free Press Journal](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra) |
| 17 Oct | Report received (20:06 UTC to Mozilla, forwarded by the Chrome Root Program). Chrome blocks the intermediates' SPKI hashes by **CRLSet**; Firefox blocks them by **OneCRL**. Neither needs a browser update or restart. | Primary (Google), Secondary (timing) | [Google Security Blog](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/); [The Hacker News](https://thehackernews.com/2026/10/eight-emudhra-ca-keys-compromised.html) |
| 19 Oct | Apple blocks the intermediates. Sectigo revokes the eight AAA cross-certificates (per the pack). Android receives equivalent protection through **Google Play services** (a restart triggers it). | Secondary | [BleepingComputer](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates); [Technobezz](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android) |
| 19–20 Oct | Microsoft blocks the intermediates (20 Oct). Go ships **1.27.3 and 1.26.10** (19 Oct UTC, verified below), which reject any chain containing the eight keys; programs must be **rebuilt**, and Go 1.25 and older receive no fix. Mozilla NSS 3.131.1 removes emSign ECC Root CA - G3. Ubuntu ships its ca-certificates fix (20 Oct, verified below). | Secondary, partly verified | [Free Press Journal](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra); [Rescana](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026); [oss-security](https://www.openwall.com/lists/oss-security/2026/10/20/30) |
| 20–21 Oct | **Pending:** Node.js, Bun, Debian, FreeBSD. Python certifi and Rust webpki-roots fix by *removing the root*, which also breaks legitimate certificates under it. macOS's built-in `curl` still accepted the affected chain after Apple's block. | Secondary | [Rescana](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026); [lilting.ch test](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys); [CSO Online](https://www.csoonline.com/article/4238338/google-mozilla-revoke-emudhra-intermediates-after-private-key-compromise.html) |
| 21 Oct | No confirmed malicious use. One recovered key has been published on GitHub. Google reports no TLS certificate chaining to the six intermediates in CT logs before the block, which protects CT-enforcing browsers but tells us nothing about forgeries aimed at clients that don't check CT. | Secondary | [BleepingComputer](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates); [iTnews](https://www.itnews.com.au/news/cloudflare-warns-fixing-botched-emudhra-certs-goes-beyond-browsers-629804) |
| 23 Oct | Oracle updates its JDK advisory: additional action is needed even on listed builds (per the pack, R4). | Per pack | [Oracle](https://ops.java/emudhra) |
| **27 Oct (Tue, morning PDT)** | **Chrome removes emSign ECC Root CA - G3 from the Chrome Root Store.** Legitimate certificates chaining to it stop working in Chrome. eMudhra's initial estimate is about 30,000 certificates needing migration. | Primary (Google), Secondary (estimate) | [Google Security Blog](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/) |
| — | AAA Certificate Services had its web (TLS) trust bit turned off in NSS in 2025, so the AAA cross-sign route matters mainly for **Windows, Java and older or vendor bundles**, not for current Mozilla-derived bundles. | Secondary (mailing-list post), consistent with the certifi check below | [oss-security](https://openwall.com/lists/oss-security/2026/10/21/4) |

**Checked directly in the review environment (26 October).** These come from
package contents, not press reports:

| Package | Finding | How checked |
|---|---|---|
| Python **certifi** (PyPI) | 2026.7.22 contains emSign ECC Root CA - G3; **2026.10.18 and every later release do not**. Release-to-release churn affects other emSign roots too: 2026.10.18 removed all four emSign roots, 2026.10.19 restored G1, C1 and ECC C3, and **2026.10.23 keeps only emSign Root CA - G1**. AAA Certificate Services is absent from all four releases checked. | Downloaded each wheel from PyPI and read `cacert.pem` labels |
| **Ubuntu 24.04 LTS** ca-certificates | `20260816ubuntu0.24.04.1` (noble-security, 20 Oct 2026, LP #2171507): "Blacklist … emSign ECC Root CA - G3 due to intermediate cert compromise". | Package changelog |
| **Node.js 22.22.0** | The built-in root list **still contains** emSign ECC Root CA - G3 (and three other emSign roots). `--use-system-ca` is available in this build. | `tls.rootCertificates` parsed with `X509Certificate` |
| Rust **webpki-roots** (crates.io) | 1.0.9 (18 Jul) contains emSign ECC Root CA - G3; **1.0.10 (19 Oct) and 1.0.11 (23 Oct) do not**. 1.0.11 removes further emSign roots, the same pattern as certifi. | Downloaded each crate and searched `lib.rs` |
| **Go** toolchain | go1.27.3 and go1.26.10 exist on the Go module proxy, published 19 Oct 22:20 UTC. *What the fix contains* is from secondary reporting. | `proxy.golang.org` toolchain index |

The certifi churn matters for availability: pinning certifi 2026.10.23 also
removes emSign Root CA - C1 and emSign ECC Root CA - C3, so partners using
certificates under those roots will break too (PB-13).

**Recommendation:** add a dated "facts as at" box to the README, with a named
owner who rechecks it daily while the incident is open. The workbook's
**Facts** sheet holds this table with a "last checked" column.

---

## 3. Does the risk tiering make sense?

**Verdict: the principle is right; the categories need a second axis.** The
pack's best insight is that an internal batch server sending customer data to a
SaaS API can matter more than a public website. Keep it. But each tier mixes
two questions, *what is the impact?* and *is this verifier still exposed?*, and
for this incident the second question decides almost everything.

### 3.1 What works

- Ranking by certificate-validation role, with "internet exposure alone does
  not determine priority".
- Treating each TLS leg separately (reverse proxy front and back; host agents
  apart from the served site).
- "Unknown is not low"; "certificate found is not proof of exploitability"; "a
  clean package scan is not proof of protection".
- Correctly noting that inbound client-certificate (mTLS) authentication needs
  no network position, while server impersonation does.
- Keeping the exposure tier and remediation status as separate fields.

### 3.2 What does not work

| # | Problem | Effect | Fix |
|---|---|---|---|
| T1 | **Tier 2 combines the safest and the most exposed populations.** "General-purpose desktops/browsers" (vendor-protected 17–20 Oct, no user action) share a tier with "ordinary outbound API clients" (no revocation checking, no CT, often a frozen bundle). | Effort goes on desktops that are already protected, while non-browser API clients wait behind them in the same queue. | Take vendor-protected browser and OS-store paths out of the tier queue and into a **coverage-assurance stream** under the §4 proviso. Tier the non-browser verifiers. |
| T2 | **No susceptibility axis.** Tiers rank impact. Exploitability depends on whether the verifier (A) gets vendor-pushed revocation or enforces CT, (B) reads an OS store that is now patched, (C) uses its own bundle (certifi, Node's built-in roots, Java cacerts, compiled Go/Rust), or (D) is embedded or vendor-controlled. | A Tier 3 internal Go service on a frozen bundle can be more exploitable than a Tier 1 path on a patched OS store, and the model can't express that. | **Priority = Impact × Verifier susceptibility** (matrix below). Keep "Tier 1–4" as the impact label so existing records still map. |
| T3 | **Availability is a footnote.** The "availability flag" appears in one paragraph near the end. | The most time-critical event this week, Chrome's 27 Oct root removal, has no owner or date. Root removal in certifi, NSS and webpki-roots will also break partners' legitimate emSign ECC chains. | Run a separate **availability track** with its own owner, register and date ([PB-13](playbooks/PB-13-our-certificates-and-availability.md)). |
| T4 | **Code signing has no tier.** It is "assessed centrally" in one sentence. | Publisher-based allow-listing, EDR reputation and software-distribution trust could all have accepted forged signatures while Windows trusted the chain. | Make application control and software distribution **Tier 1**, add a retrospective signer hunt ([PB-12](playbooks/PB-12-code-signing-and-application-control.md)). |
| T5 | **Tier 1 examples omit the paths that turn impersonation into authentication bypass or code execution.** | Teams don't recognise OIDC discovery/JWKS fetches, SAML metadata refresh, package and update downloads, VPN/ZTNA clients, secrets-manager calls and token-bearing webhooks as Tier 1. | Name them explicitly ([PB-16](playbooks/PB-16-identity-and-mtls.md)). |
| T6 | **The TLS-inspection proxy is listed as one Tier 1 example among many.** | For inspected traffic, the proxy's upstream verifier *is* the verifier for every client behind it, browsers included. A browser's CRLSet never sees the real chain, because the proxy re-signs it. | Treat the inspection proxy as the **first** Tier 1 item, configured to *block* untrusted or revoked upstream certificates ([PB-09](playbooks/PB-09-proxies-gateways-and-tls-inspection.md)). |
| T7 | **No target dates.** "This pack sets no independently approved SLA." | Without a reference date, "agree a date with the service owner" tends to slip. Management can't tell late from on-track. | Propose **indicative windows** for the incident lead to approve (workbook decision D-03). |

### 3.3 Proposed prioritisation matrix

Impact keeps the pack's tier language: H = Tier 1 impact (credentials, sensitive
data, privileged action, many downstream systems); M = Tier 2/3; L = Tier 4 or
low consequence.

| Verifier susceptibility → | **A** Vendor-pushed block or CT enforced (browsers, OS store already updated) | **B** OS store consumer; update available but not yet confirmed | **C** Own bundle or compiled roots (certifi, Node, Java cacerts, Go, Rust, containers) | **D** Embedded, appliance or vendor-controlled; no fix yet | **?** Unknown |
|---|---|---|---|---|---|
| **H** | P3 — confirm coverage | **P1** | **P1** | **P1** + exception and compensating control | **P1 to assess** |
| **M** | P4 — confirm coverage | P2 | P2 | P2 + vendor tracking | P2 to assess |
| **L** | P4 | P3 | P3 | P3 + vendor tracking | P3 to assess |

**Proposed indicative windows** (decision D-03, not yet approved): P1 contain
within 72 hours and close within 7 days; P2 close within 14 days; P3 within 30
days; P4 confirm in the normal cycle. Escalate suspected exploitation
immediately, whatever the priority. Overdue work keeps its original date when
reassigned, as the pack already says.

### 3.4 Changes to the worked examples

| Pack scenario | Pack says | Proposed |
|---|---|---|
| Ordinary user browsing the internet | Tier 2 assessment | Impact M × Susceptibility A → **P4, covered by the browser proviso** (§4), unless browsing goes through TLS inspection, in which case it inherits the proxy's status. |
| Internal low-impact service accepting public roots | Tier 3 | Unchanged if it uses a patched OS store (P3). **P2** if it uses a bundled or compiled store. |
| Container with outbound external API calls | Tier 1 or 2 by impact | Unchanged, plus: susceptibility is usually **C** because the image carries its own CA bundle and is only fixed by a rebuild. |
| *(new)* API validating JWTs from an external IdP | — | **P1** (H × B/C). A forged TLS certificate for the JWKS URL lets an attacker mint tokens the API accepts. |
| *(new)* Self-hosted artifact proxy (Artifactory, Nexus) fetching from public registries | — | **P1**. A chokepoint for every build; usually Java, so susceptibility C. |
| *(new)* Endpoint with publisher-rule application control | — | **P1** for the policy owner. Forged code signatures could satisfy publisher-name rules while Windows trusted the chain. |
| *(new)* Public website whose own certificate chains to emSign ECC Root CA - G3 | Tier 4 (server posture) | Tier 4 for security, **availability flag = Yes**; replace before or soon after 27 Oct. |

---

## 4. The browser and desktop proviso

**Recommendation: yes, and make it explicit wherever the pack says browsers
are protected.** Mainstream browsers and OS trust stores *were* protected by
the vendors, but only under conditions an enterprise can break. Proposed text
for the README executive summary and the Desktop/browser row of the action
plan:

> **Browser and desktop proviso.** Mainstream browsers and operating-system
> trust stores received vendor protection between 17 and 20 October 2026
> without user action: Chrome (CRLSet) and Firefox (OneCRL) on 17 October,
> Apple on 19 October, Microsoft on 20 October, and Android through Google Play
> services. We record a managed browser or OS configuration as **Validated**,
> not "fixed", and only on a coverage report showing **all** of the following:
>
> 1. **Reachability.** Devices have been online since 20 October and can reach
>    the vendors' update services. No enterprise policy or egress rule blocks
>    browser component updates, Mozilla Remote Settings, or Windows automatic
>    root and disallowed-certificate updates.
> 2. **Currency.** The browser and OS are supported versions. Android devices
>    have current Google Play services.
> 3. **No interception.** The traffic is not decrypted by a TLS-inspection
>    proxy. Where it is, the proxy's upstream verifier makes the trust decision,
>    and the browser's protection does not apply to that traffic.
> 4. **Image hygiene.** Non-persistent VDI images, kiosks, golden images and
>    offline or air-gapped devices have been refreshed or confirmed separately.
>
> The proviso covers **the browser and the OS trust store only**. It does not
> cover other software on the same device: command-line tools (macOS's
> built-in `curl` was shown to accept the affected chain after Apple's block),
> Java, Python, Node and Go applications, Electron apps, VPN and ZTNA clients,
> backup and management agents, or developer toolchains. These belong to the
> long tail (§5).
>
> On **27 October** Chrome also removes emSign ECC Root CA - G3 from its root
> store. That is an availability change: legitimate sites using certificates
> under that root will fail in Chrome. It is not a new security gap.

**Status language.** Use "vendor protection delivered on <date>; enterprise
coverage confirmed for <n>% of managed devices as at <date>". Avoid "has been
fixed" until the coverage report exists, and say "is being fixed" only for
items with a named owner and date.

**Evidence that satisfies the proviso** (detail in
[PB-01](playbooks/PB-01-browsers-and-desktop-os.md)): one representative test
per managed configuration (for example Windows + Chrome, Windows + Edge,
Firefox ESR, macOS + Safari, the VDI image, Android, iOS), a check that egress
to vendor update endpoints is allowed, a check of the Windows root auto-update
policy, and the device-management coverage percentage. The workbook's
**Browser & desktop** sheet computes "proviso met" from these inputs.

---

## 5. The long tail: internet, application and certificate-dependent systems

**Agree. Reframe the response as two phases with different methods.**

| Phase | Population | Who fixed it | Our job | Method |
|---|---|---|---|---|
| **1. Vendor-protected** | Browsers, OS trust stores, managed mobile | Vendors, 17–20 Oct | Confirm coverage under the proviso | Device-management reports plus a representative test per configuration |
| **2. Long tail** | Every system that validates a public-CA certificate with **its own** trust data or verifier | Us, or our suppliers | Find, fix or contain, prove | Egress-led discovery, playbooks, vendor tracking, exceptions |

### 5.1 What is in the long tail

| Category | Typical examples | Why it is exposed | Playbook |
|---|---|---|---|
| Server-side outbound clients | Apps calling SaaS, payment, partner or IdP APIs; webhooks; batch transfers | Runtime bundles rarely check revocation or CT | PB-04 to PB-07 |
| Runtimes with their own bundle | Java cacerts, Python certifi, Node built-in roots, Go and Rust compiled roots | Only fixed when the bundle, runtime or binary is updated | PB-04 to PB-07 |
| Containers and serverless | Base images, distroless and scratch images, Lambda/Functions packages with vendored certifi | Fixed only by rebuild and redeploy; scanners may not flag | PB-08 |
| CI/CD and supply chain | Build runners, package managers, artifact proxies | A forged certificate here becomes code execution | PB-08 |
| Desktop non-browser software | Electron apps, CLI tools, developer toolchains, agents, VPN clients | Outside the browser proviso | PB-01, PB-06, PB-11 |
| Network and security appliances | TLS-inspection proxies, WAF, load balancers, API gateways, SIEM collectors | Own trust stores; vendor-controlled updates | PB-09, PB-11 |
| OT, IoT and facilities | Building management, printers, cameras, UPS with cloud management | Firmware-embedded roots; slow vendors | PB-11 |
| In-house mobile apps | Apps with custom trust anchors, bundled TLS stacks or pinning | Not fixed by the OS update | PB-10 |
| Suppliers and SaaS | Processors and SaaS platforms holding our data or calling our APIs | Outside our control | PB-11 |
| Inbound mTLS | Partner APIs accepting client certificates under public roots | No network position needed to exploit | PB-16 |

### 5.2 How to find it without a CBOM

The pack rightly avoids requiring a CBOM. "Use existing inventories" is not
enough on its own, because CMDBs rarely record which systems make outbound TLS
validation decisions. The best discovery source already exists:

1. **Egress telemetry.** Pull 30 days of outbound connections on TCP 443 (and
   other TLS ports) from the forward proxy, firewall, NAT and cloud egress logs,
   grouped by source host or workload. Every long-tail system that matters
   appears here.
2. **Map sources to owners** through the CMDB or cloud tags. Each source becomes
   a long-tail record in the workbook.
3. **Treat direct-to-internet TLS that bypasses the proxy as a finding.** It is
   the population no shared control protects.
4. **Add software inventory** (Intune, SCCM, Jamf) for desktop Java runtimes,
   Electron apps and agents, and the **container registry** for base-image
   lineage.
5. **Where TLS inspection logs record the upstream certificate issuer**, look
   for destinations whose chain includes eMudhra or emSign. That gives both
   the hunt (PB-14) and the partner availability list (PB-13).

### 5.3 How the tail closes

The tail will not close on one date. The pack should state a closure policy:

- **Cut-off date** (decision D-09), after which an unremediated long-tail item
  needs a formal exception.
- **Approved compensating controls** for exceptions: restrict egress to known
  destinations; route through a remediated inspection proxy that blocks
  untrusted upstream chains; pin the expected CA or key for critical
  integrations; segment the network.
- **Vendor-dependent items** close against the vendor's fix with a review date
  tied to the vendor's ETA. "Vendor has not responded" is tracked, not
  accepted.
- **Suppliers**: ask the critical ones (payments, identity, SaaS holding
  regulated data) for written confirmation, using the template in PB-11.

---

## 6. Enterprise gaps and improvements

Each finding has a severity and a suggested disposition. Severity reflects the effect on the
enterprise's risk or on its ability to execute the response.

| ID | Pack location | Challenge | Severity | Evidence or "opinion" | Suggested disposition |
|---|---|---|---|---|---|
| G1 | README, all | No vendor actions, dates or fix status anywhere in the pack. "Already-protected browsers" is asserted without saying by what or since when. Several remediations (Node, Debian) were pending on 24 Oct and should be tracked. | High | §2 sources | Add a dated facts box and owner; use the workbook **Facts** sheet. |
| G2 | RISK_TIERS, ACTION_PLAN | Chrome's **27 Oct** root removal is not mentioned. Our own certificates, or partners' certificates, under emSign ECC Root CA - G3 break in Chrome, and later in any client whose bundle removes the root. | High (time-critical) | [Google](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/) | Availability track and [PB-13](playbooks/PB-13-our-certificates-and-availability.md); service desk KB ([PB-15](playbooks/PB-15-service-desk.md)). |
| G3 | ACTION_PLAN (SOC row) | **No retrospective hunt.** The keys were recoverable from generation (30 Apr 2025) to the blocks (17–20 Oct 2026), up to 535 days. Because eMudhra says these intermediates issued no end-entity certificates, *any* certificate chaining to them is presumptively forged, so a hit means an incident rather than one more finding to triage. That makes the hunt cheap and decisive. | High | §2; opinion on hunt value | Add [PB-14](playbooks/PB-14-soc-threat-hunt.md): EDR signer search, inspection-proxy and NDR issuer search, retention limits recorded as a documented gap. |
| G4 | ACTION_PLAN, CBOM | **Code signing is one sentence.** Two code-signing intermediates were reportedly trusted by Microsoft. Publisher-name allow-listing rules and EDR reputation are the exposure. | High | [Free Press Journal](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra) | [PB-12](playbooks/PB-12-code-signing-and-application-control.md); Tier 1 for application-control owners. |
| G5 | RISK_TIERS, VALIDATION | **TLS inspection** is covered in the validation appendix but not connected to the browser position. Behind an inspection proxy, browser protections don't apply; the proxy decides. Conversely, one proxy fix protects every inspected client, including non-browser ones. | High | Opinion (architecture) | Proviso condition 3; make the proxy the first Tier 1 item; [PB-09](playbooks/PB-09-proxies-gateways-and-tls-inspection.md). |
| G6 | All | **Findability.** A service owner has to read five documents to learn what to do for one stack. REMEDIATION_OPTIONS is ordered by vendor mechanism, not by team or technology. Teams can't see whether a section applies to them. | High (execution) | Opinion | [`playbooks/`](playbooks/README.md) indexed by team and by stack, with an "applies if" test at the top of each. |
| G7 | RISK_TIERS | Tiering lacks a susceptibility axis (see §3). | Medium | Opinion | Adopt the §3.3 matrix (D-01). |
| G8 | ACTION_PLAN | **Governance.** No named incident lead, decision authority, decision log, executive reporting cadence, or criteria for closing the incident as a whole. | Medium | Opinion | Workbook **Decisions** sheet (D-01 to D-16), including D-14 closure criteria. |
| G9 | ACTION_PLAN | **No target dates.** | Medium | Opinion | D-03 indicative windows. |
| G10 | Absent | **Mobile** is absent: managed Android (Play services delivery), devices without Google Play services, iOS, and in-house apps with custom trust anchors, bundled TLS stacks or pinning. | Medium | [Technobezz](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android) | [PB-10](playbooks/PB-10-mobile.md). |
| G11 | RISK_TIERS | **Identity paths** (OIDC discovery/JWKS, SAML metadata refresh, OAuth token endpoints) are not named, yet they turn TLS impersonation into token forgery or credential theft. | Medium | Opinion (well-known pattern) | [PB-16](playbooks/PB-16-identity-and-mtls.md); add to Tier 1 examples. |
| G12 | ACTION_PLAN | **Suppliers**: "obtain applicable guidance" is passive. There is no list of critical suppliers, standard question, or tracking. | Medium | Opinion | PB-11 template; workbook **Long tail** sheet with vendor status. |
| G13 | Absent | **Service desk.** No knowledge article for "is my PC safe?" or for site failures from 27 Oct, and no instruction never to tell users to click through certificate errors. | Medium | Opinion | [PB-15](playbooks/PB-15-service-desk.md). |
| G14 | Absent | **Legal and regulatory.** No trigger for notification if exploitation is found. For a Hong Kong entity with UK affiliates this means the relevant HK and UK regulators and privacy authorities; Legal must confirm which apply. Exposure alone is unlikely to trigger notification, but that position should be recorded. | Medium-Low | Opinion; obligations need Legal confirmation | D-11. |
| G15 | Absent | **CI/CD and artifact proxies** are mentioned only as "software-update/build pipelines". Self-hosted Artifactory and Nexus are chokepoints that are usually Java-based. | Medium | Opinion | [PB-08](playbooks/PB-08-containers-kubernetes-and-cicd.md). |
| G16 | CBOM | The AAA route note doesn't say *where* AAA still matters (Windows, Java, older or vendor bundles). Teams on current Mozilla-derived bundles may over-test. | Low | [oss-security](https://openwall.com/lists/oss-security/2026/10/21/4) (verify) | Noted in each playbook's "watch out" section. |
| G17 | All | Repeated "not universally required" caveats in every document weaken the required actions. Readers can't tell the mandatory from the optional. | Low | Opinion | Playbooks put *required* steps first; optional depth links back to the pack. |
| G18 | VALIDATION | Testing depends on the external test site, which enterprise egress policy may block. A proxy block page is inconclusive for an app but a **pass** for the proxy itself. | Low | Pack's own table | [PB-00](playbooks/PB-00-test-and-evidence.md) clarifies; D-13 internal test service option. |
| G19 | TRACKER | Reporting counts lack a definition of done and headline KPIs (Tier 1 open, unknowns, browser coverage %, vendor-pending count, exceptions expiring). | Low | Opinion | Workbook **Dashboard**. |

---

## 7. Making the technical remediation findable

The pack's technical content is accurate and careful. The problem is
navigation. The playbooks reorganise the same guidance, plus the additions
above, so a team can find and act on its part in a few minutes:

- **Two entry points** in [`playbooks/README.md`](playbooks/README.md): *by
  team* (desktop, Windows, Linux, app owners, containers, network, mobile, PKI,
  SOC, service desk, supplier managers) and *by what you run* (Java, Python,
  Node/Electron, Go/Rust/PHP/Ruby, .NET, containers, proxies, appliances).
- **The same six headings in every playbook:** *Applies if* (yes/no test) →
  *Fix* (options in order of preference) → *Verify* → *Record* (maps to the six
  tracker fields) → *Watch out for* → *Escalate when*.
- **Copy-paste checks** that are read-only and self-verifying (they print what
  the runtime actually uses), with placeholders clearly marked.
- **Playbook codes** (PB-00 to PB-16) that the workbook's tracker uses in a
  dropdown, so each record points to the steps used.
- **Depth stays in the pack.** Each playbook links back to the relevant section
  of REMEDIATION_OPTIONS or APPLICATION_VALIDATION rather than duplicating
  every caveat.

The playbooks are illustrative procedures. None of the commands have been run
against an enterprise estate. Check each one against the exact deployed
version and the current vendor advisory before use.

---

## 8. Decisions to capture

The workbook's **Decisions** sheet carries these as open items, each with
options and a recommendation. None has been decided.

| ID | Decision | Recommendation |
|---|---|---|
| D-01 | Prioritisation model | Adopt impact × verifier susceptibility (§3.3); keep tier labels for impact |
| D-02 | Browser/desktop proviso | Accept vendor protection as closure only with the §4 coverage report |
| D-03 | Target windows | Approve indicative P1–P4 windows |
| D-04 | Our certificates under emSign ECC Root CA - G3 | Replace now; owner PKI; no extension beyond 27 Oct for external sites |
| D-05 | Root removal in managed stores | Accept partner availability impact; time-boxed exceptions only |
| D-06 | AAA route on Windows and Java | Disallow the specific intermediates; remove AAA only where the vendor prescribes it |
| D-07 | TLS-inspection upstream policy | Block untrusted or revoked upstream certificates; no "warn" or "allow" |
| D-08 | Retrospective hunt scope | Run PB-14 hunts over available retention; record the gap to 30 Apr 2025 |
| D-09 | Long-tail cut-off | Set a date; exceptions need compensating controls |
| D-10 | Supplier assurance | Formal request to critical suppliers only |
| D-11 | Regulatory stance | No notification for exposure alone; Legal defines triggers on evidence of exploitation |
| D-12 | Reporting | Unit = service; executive update twice weekly until P1 is closed |
| D-13 | Internal test service | Stand one up if more than a few P1 systems lack test-site egress |
| D-14 | Incident closure criteria | P1 and P2 closed or excepted; proviso met; hunts complete; availability track closed |
| D-15 | In-house mobile apps | Review trust configuration of every app that handles credentials or payments |
| D-16 | Staff communications | No all-staff action; service desk KB only |

---

## 9. Sources

Primary pages could not be fetched from the review environment; links are given
for the response lead to confirm.

- Google Security Blog: [Protecting Chrome and Android users from compromised eMudhra intermediate CA certificates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
- Cloudflare: [8 eMudhra certificates are compromised. Here's what you need to know](https://blog.cloudflare.com/emudhra-certificate-compromise/)
- Mozilla: [Protecting Firefox users following the compromise of eMudhra intermediate certificates](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/)
- Sectigo: [Advisory on compromised eMudhra intermediate certificates](https://www.sectigo.com/resource-library/advisory-on-compromised-emudhra-intermediate-certificates)
- BleepingComputer: [Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
- CSO Online: [Google, Mozilla revoke eMudhra intermediates after private key compromise](https://www.csoonline.com/article/4238338/google-mozilla-revoke-emudhra-intermediates-after-private-key-compromise.html)
- iTnews: [Cloudflare warns fixing botched eMudhra certs goes beyond browsers](https://www.itnews.com.au/news/cloudflare-warns-fixing-botched-emudhra-certs-goes-beyond-browsers-629804)
- The Hacker News: [Eight eMudhra CA keys compromised](https://thehackernews.com/2026/10/eight-emudhra-ca-keys-compromised.html)
- Free Press Journal: [Why did Apple, Google, Microsoft and Mozilla block certificates from eMudhra?](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra)
- Technobezz: [Google blocks compromised eMudhra keys in Chrome and Android](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android)
- Rescana: [eMudhra intermediate CA key compromise, October 2026](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026)
- Endor Labs: [The trust stores you patched, and the ones you didn't know you had](https://www.endorlabs.com/learn/emudhra-intermediate-ca-compromise-the-trust-stores-you-patched-and-the-ones-you-didnt-know-you-had)
- lilting.ch: [Recovering eMudhra's weak P-384 CA keys and testing curl, Node.js and uv on macOS](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
- oss-security: [20 Oct thread](https://www.openwall.com/lists/oss-security/2026/10/20/30); [21 Oct (AAA trust bit)](https://openwall.com/lists/oss-security/2026/10/21/4)
- Veeam: [KB4952 support statement](https://www.veeam.com/kb4952)
- Business Today: [MeitY says Indian DSC and eSign unaffected](https://www.businesstoday.in/technology/story/impact-has-been-very-limited-meity-on-emudhra-security-incident-says-indian-dsc-and-esign-unaffected-562927-2026-10-21)
