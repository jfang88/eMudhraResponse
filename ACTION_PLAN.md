# Enterprise action plan

**Version 2 · 26 October 2026 · Operational instruction · Proposed: decisions pending ([Appendix A](#appendix-a-decision-register))**

## 1. Purpose

Answer four questions for every service and shared component: **Are we
exposed? Who owns the fix? What did we do? What evidence reasonably supports
closure?**

This plan sets how the response runs. [Prioritisation](RISK_TIERS.md) sets
the order of work. The [completion tracker](COMPLETION_TRACKER.md) sets what
each owner returns. The [playbooks](playbooks/README.md) set the technical
steps for each team.

## 2. Governance

| Role | Responsibility |
|---|---|
| **Incident lead** | Runs the response, owns the decision register and the facts box in the [README](README.md#situation-as-at-27-october-2026), chairs daily stand-ups while P1 items are open |
| **Response steering** (incident lead, Security/PKI, platform, network, application and service-desk leads) | Approves operational decisions; resolves cross-team blockers |
| **Risk committee / CISO** | Approves target windows, exception policy, closure criteria and the regulatory stance |
| **Security / PKI** | Owns prioritisation, the approved test matrix, the certificate reference and exception review |

Decisions are recorded in [Appendix A](#appendix-a-decision-register) and in
the workbook's *Decisions* sheet (`eMudhra-response-workbook.xlsx`).

**Incident closure criteria** (approval is decision D-14): all P1 and P2
records closed or under an approved exception; the browser and desktop
proviso met for every managed configuration; hunts H1–H6 complete, with
retention gaps recorded; the availability track closed. Remaining P3 and P4
items then move to business-as-usual vulnerability management.

## 3. How the response runs

| Workstream | Population | Method | Section |
|---|---|---|---|
| **Phase 1: confirm vendor protection** | Managed browsers, OS trust stores, managed mobile | Coverage report under the browser and desktop proviso | [§5](#5-phase-1-browser-and-desktop-proviso) |
| **Phase 2: the long tail** | Everything that validates public certificates with its own trust list or verifier | Egress-led discovery, playbooks, vendor tracking, exceptions | [§6](#6-phase-2-the-long-tail) |
| **Availability track** | Certificates we present, and partner endpoints we call, under emSign roots being removed | Find, replace, notify partners | [§7](#7-availability-track-tuesday-27-october) |
| **Retrospective hunt** | Evidence of forged certificates or signatures since April 2025 | Targeted hunts H1–H6 | [§8](#8-retrospective-hunt) |

**Fix the chokepoints first:** TLS-inspection and egress proxies, identity
integrations, build pipelines and artifact proxies. Each protects many
downstream systems. The full list is in
[Prioritisation §5](RISK_TIERS.md#5-fix-first-paths-that-turn-a-forged-certificate-into-something-worse).

## 4. Minimum response for every service

1. **Scope and owner.** Identify the service or managed asset group and one
   accountable owner. Use existing inventories, configuration records and
   egress telemetry (§6). Don't start a complete cryptographic inventory.
2. **Prioritise.** Record impact (H/M/L) and verifier susceptibility (A–D or
   Unknown); the matrix gives the priority
   ([Prioritisation §4](RISK_TIERS.md#4-priority-matrix)). Note whether the
   service uses an assured standard platform or a custom trust configuration.
3. **Fix.** Follow the team's [playbook](playbooks/README.md) and apply the
   smallest effective remedy (§11).
4. **Close.** Choose a closure route below and return the
   [six-field record](COMPLETION_TRACKER.md).
5. **Escalate** credible critical exposure, suspected exploitation, outages
   and unresolved blockers through the existing incident and change
   processes.

### Three closure routes

| Route | When appropriate | Minimum evidence |
|---|---|---|
| **A: Standard platform assurance** | The application uses a centrally maintained, remediated platform or store and has no relevant trust overrides | Shared assurance reference; evidence the deployment is covered; owner confirmation that no relevant custom store, verifier or override is used |
| **B: Application-specific validation** | Custom bundles, embedded runtimes, uncertain behaviour, or critical paths not adequately covered by Route A | Applicable fix or configuration and activation evidence; representative validation through the relevant verifier ([PB-00](playbooks/PB-00-test-and-evidence.md)); normal integration checks. Link existing records |
| **C: Not applicable to the assessed role** | Serve-only process with an unaffected presented chain, or private-CA-only validation without public-trust fallback | Short owner statement defining scope, supported by existing configuration or architecture evidence. Refer separately owned dependencies to their owners |

P1 needs strong assurance of coverage, but not duplicate testing where Route
A demonstrably covers the critical path. Route C can close without an
incident-specific change. Unknown exposure is not a valid Route C rationale.

### Shared assurance and proportionate testing

Security/PKI and platform owners publish an **assurance record** for each
materially different standard configuration: the vendor remedy applied,
representative validation covering the emSign route (and the AAA route where
AAA is still trusted), deployment coverage, known exclusions and normal
service checks. A package compliance report alone does not establish shared
assurance.

Applications reference that record rather than reproduce it. Existing
deployment reports can show that fleet members match the assessed
configuration. No per-user screenshots, per-replica tests or per-application
certificate hashes are needed for an unchanged assured configuration.

Additional checks are triggered by a different runtime or verifier, a custom
store or mounted bundle, a validation override, an identity-dependent policy
or a materially different network validation path. Test the difference, not
every unrelated component.

Don't open production internet access just to reach the external test site.
Use offline verification or a representative environment with documented
equivalence (PB-00; decision D-13 covers an internal test service).
Product- and version-specific vendor evidence can support closure where
direct tests are impractical, provided applicability, deployment and
limitations are recorded and Security accepts the residual uncertainty.

## 5. Phase 1: browser and desktop proviso

Browsers and OS trust stores received vendor protection between 17 and 20
October without user action: Chrome (CRLSet) and Firefox (OneCRL) on 17
October, Apple on 19 October, Android through Google Play services, and
Microsoft on 20 October (the Windows Disallowed list, plus a CRLSet update for
Edge and WebView2). Microsoft then removed seven related certificates from the
Windows trust list on 26 October and from Edge on 27 October. We record a managed browser or OS configuration as
**Validated**, not "fixed", and only on a coverage report showing **all**
of the following (approval is decision D-02):

1. **Reachability.** Devices have been online since 20 October and can reach
   vendor update services. No policy or egress rule blocks browser component
   updates, Mozilla Remote Settings, or Windows automatic root and
   disallowed-certificate updates. Disconnected or locked-down Windows machines have the
   26 October out-of-band update installed from the Microsoft Update Catalog.
2. **Currency.** Supported browser and OS versions; Android Google Play
   services current.
3. **No interception.** The traffic is not decrypted by a TLS-inspection
   proxy, or the proxy has passed
   [PB-09](playbooks/PB-09-proxies-gateways-and-tls-inspection.md). Behind
   inspection, the proxy makes the trust decision and browser blocks don't
   apply.
4. **Image hygiene.** Non-persistent VDI images, kiosks, golden images and
   long-offline devices are refreshed or checked separately.

The coverage threshold for "Validated" is proposed at 95% of in-scope devices
(D-02). Evidence is one representative test per managed configuration plus
the device-management coverage report. No testing request goes to every
user.

**The proviso covers the browser and the OS trust store only.** It does not
cover other software on the same device: command-line tools (macOS's built-in
`curl` was shown to accept the affected chain after Apple's block), Java,
Python, Node and Go applications, Electron apps, VPN and ZTNA clients,
management agents, or developer toolchains. These belong to Phase 2.

**Status language:** "vendor protection delivered on <date>; enterprise
coverage confirmed for <n>% of managed devices as at <date>". Steps:
[PB-01](playbooks/PB-01-browsers-and-desktop-os.md) (desktops) and
[PB-10](playbooks/PB-10-mobile.md) (mobile).

## 6. Phase 2: the long tail

Once Phase 1 is confirmed, the remaining risk sits in systems that validate
public certificates with their own trust data or verifier:

| Category | Examples | Playbook |
|---|---|---|
| Server-side outbound clients | Apps calling SaaS, payment, partner or IdP APIs; webhooks; batch transfers | PB-04 to PB-07, PB-16 |
| Runtimes with their own bundle | Java `cacerts`, Python certifi, Node built-in roots, Go and Rust compiled roots | PB-04 to PB-07 |
| Containers and serverless | Base images, distroless and scratch images, function packages with vendored bundles | PB-08 |
| CI/CD and supply chain | Build runners, package managers, artifact proxies | PB-08 |
| Desktop non-browser software | Electron apps, CLI tools, developer toolchains | PB-01, PB-06 |
| Network and security appliances | Inspection proxies, WAF, load balancers, API gateways | PB-09, PB-11 |
| VPN, ZTNA and agents | Endpoint, backup, monitoring and management agents | PB-11 |
| OT, IoT and facilities | Building management, cameras, printers, UPS with cloud management | PB-11 |
| In-house mobile apps | Custom trust anchors, bundled TLS stacks, pinning | PB-10 |
| Suppliers and SaaS; inbound mTLS | Processors holding our data; partner APIs accepting client certificates | PB-11, PB-16 |

**Find it without a CBOM.**

1. **Egress telemetry.** Pull 30 days of outbound TLS connections from forward
   proxy, firewall, NAT and cloud egress logs, grouped by source host or
   workload.
2. **Map sources to owners** through the CMDB or cloud tags. Each source not
   already covered by a platform record becomes a long-tail record (workbook
   *Long tail* sheet).
3. **Treat direct-to-internet TLS that bypasses the proxy as a finding.** No
   shared control protects it.
4. **Add inventories:** software inventory (Java runtimes, Electron apps,
   agents), the container registry (base-image lineage) and the supplier
   register.
5. **Where inspection logs record the upstream issuer**, list destinations
   whose chain includes eMudhra or emSign. This feeds §7 and §8.

**How the tail closes.**

- **Cut-off date** (decision D-09, proposed 25 November 2026). After it, any
  unremediated long-tail item needs an approved exception.
- **Approved compensating controls:** restrict egress to known destinations;
  route through a remediated inspection proxy that blocks untrusted upstream
  chains; pin the expected CA or key for a critical integration; segment the
  network.
- **Vendor-dependent items** close against the vendor's fix, with a review
  date tied to the vendor's ETA. "Vendor has not responded" is tracked, not
  accepted.
- **Suppliers:** written confirmation from critical suppliers only (payments,
  identity, SaaS with regulated data, managed security providers; decision
  D-10), using the template in
  [PB-11](playbooks/PB-11-vendor-products-appliances-and-suppliers.md).

## 7. Availability track (Tuesday 27 October)

On the morning of 27 October (US Pacific time), Chrome removes **emSign ECC
Root CA - G3** from the Chrome Root Store, and Microsoft removes seven related
certificates from the Edge trust list (already removed from the Windows trust
list on 26 October). Legitimate certificates chaining to them stop working in
Chrome, Edge and Windows clients; eMudhra estimates about 30,000 certificates
need migration worldwide. On 22 October eMudhra also announced it is leaving
the global SSL/TLS certificate business, offering free replacements through
partners, so **every eMudhra-issued TLS certificate we use will eventually need
replacing**, not only those under emSign ECC Root CA - G3. Mozilla-derived bundles are removing the same root:
Ubuntu `ca-certificates`, certifi from 2026.10.18 and webpki-roots from
1.0.10. Later releases drop further emSign roots (certifi 2026.10.23 keeps
only emSign Root CA - G1).

A certificate *issued to us* by eMudhra is not compromised by this incident.
It will simply stop being trusted. Owner: PKI / certificate management
([PB-13](playbooks/PB-13-our-certificates-and-availability.md)).

- **Certificates we present.** Find them with the lifecycle tool, a
  Certificate Transparency search of our domains and an endpoint scan.
  Replace those under emSign ECC Root CA - G3 now (decision D-04); plan the
  rest, given eMudhra's exit.
- **Partner endpoints we call.** Updating our trust stores breaks partners
  still using the removed roots. Notify them and agree dates. If
  business-critical traffic breaks, raise a time-boxed exception with a
  compensating control (decision D-05). **Don't restore the root globally.**
- **Service desk.** Knowledge article live before 27 October
  ([PB-15](playbooks/PB-15-service-desk.md)).

Mark affected services with the availability flag in the tracker until the
work is done.

## 8. Retrospective hunt

The keys were weak from generation on 30 April 2025, so clients were exposed
for up to about 18 months (535 days to the first vendor block), and longer for
software still unpatched. eMudhra states the affected intermediates issued
no end-entity certificates. **Any certificate or signature chaining to them,
outside known test hosts, is therefore presumptively forged**, and a single
hit is an incident. That makes a small targeted hunt cheap and decisive. It
replaces any idea of a historical investigation for every application.

| Hunt | Data source | Look for |
|---|---|---|
| H1 | EDR file-signature telemetry | Binaries signed under the two code-signing intermediates (highest value) |
| H2 | TLS-inspection / SWG logs | Upstream certificates issued by the six TLS intermediates |
| H3 | NDR / Zeek x509 | Same issuers; TLS 1.3 hides certificates from passive sensors |
| H4 | Application TLS logs on P1 integrations | Peer certificate details |
| H5 | Certificate Transparency | Our domains; Google found no certificates under the six TLS intermediates before the block |
| H6 | IdP and API logs | Unexpected JWKS key IDs or SAML signing-certificate changes |

Search what retention allows and record the gap back to 30 April 2025 as a
documented limitation (decision D-08). Exclude `*.we-have-a-problem.com`,
where staff testing produces legitimate hits. Steps:
[PB-14](playbooks/PB-14-soc-threat-hunt.md) and
[PB-12](playbooks/PB-12-code-signing-and-application-control.md).

## 9. Responsibilities and first actions

Assign one accountable owner per service or shared component. Platform teams
own their reusable evidence; application teams confirm the platform applies
to them, or use Route B where it doesn't.

| Team | Accountable for | First action (by Thu 29 Oct) | Playbooks |
|---|---|---|---|
| **Incident lead** | Running the response; decisions; facts box; reporting | Decisions D-01 to D-07; name owners for each team below | — |
| **Security / PKI** | Prioritisation, test matrix, certificate reference, exceptions | Publish the test matrix and certificate reference; replace our emSign ECC G3 certificates | PB-00, PB-13 |
| **Network / proxy / WAF** | TLS legs verified by shared infrastructure | Validate the inspection proxy; set *block* for untrusted upstream; share the issuer-log query | PB-09 |
| **IAM / API platform** | Identity integrations and inbound mTLS | Assess JWKS, SAML metadata and mTLS trust lists | PB-16 |
| **Linux and Windows platform** | Standard stores and runtimes | Publish a Route A record per build; on disconnected Windows servers, install the 26 Oct out-of-band update or distribute the disallowed certificates | PB-02, PB-03 |
| **Container / DevSecOps** | Images, pipelines, artifact proxies | Rebuild base images; fix the artifact-proxy JVM; list workloads by digest | PB-08 |
| **Desktop / EUC** | Managed browsers and desktop OS | Coverage report against the four proviso conditions; inventory non-browser apps | PB-01 |
| **Mobile / MDM** | Managed mobile fleet; in-house app trust | Fleet coverage; list in-house apps that handle credentials or payments | PB-10 |
| **Application owners** | Confirming platform applicability or custom trust | Identify runtime and trust store; P1 paths first; return the six-field record | PB-04 to PB-07, PB-16 |
| **Endpoint security / software distribution** | Code-signing trust and application control | Check publisher-based rules; confirm Windows distrust coverage | PB-12 |
| **SOC / incident response** | Hunts and monitoring | Run H1 and H2; record retention per source | PB-14 |
| **Supplier and service owners** | Vendor products, appliances, SaaS | Send requests to critical suppliers; enter the long tail in the register | PB-11 |
| **Service desk** | User contacts and triage | Knowledge article live before 27 Oct | PB-15 |
| **Legal / compliance** | Notification triggers | Confirm applicable regulators and triggers (D-11) | — |

## 10. Timing, plan and reporting

Target windows by priority are in
[Prioritisation §4](RISK_TIERS.md#4-priority-matrix) (indicative, pending
D-03). For existing work they give this plan:

| By | Milestones |
|---|---|
| **Tue 27 Oct** | D-04 and D-16 decided. Our certificates under emSign ECC Root CA - G3 replaced (external first). Service desk article live |
| **Thu 29 Oct** (72 h) | D-01, D-02, D-03, D-07 and D-12 decided (28 Oct); D-05 and D-06 (29 Oct). Inspection proxy validated. Identity paths assessed. P1 contained. Proviso coverage report. Hunts H1–H2 run |
| **Mon 2 Nov** (7 days) | P1 closed or excepted. Egress-led long-tail register built. Supplier requests sent. Hunts H3–H6 done. Remaining decisions made |
| **Mon 9 Nov** (14 days) | P2 closed or excepted. Base images rebuilt and redeployed. Vendor ETAs tracked |
| **Wed 25 Nov** (30 days) | P3 closed. Long-tail cut-off (D-09). Closure review (D-14); P4 to business as usual |

**Reporting** (decision D-12): count by priority and status, with the service
as the unit. Don't mix units or double-count shared platforms. Send daily
updates only for open P1 items, material blockers and outages; send an
executive update twice weekly until P1 is closed. Record discovery and
assignment dates for new findings, and don't reset existing overdue work when
it is reassigned. Headline indicators are listed in the
[completion tracker](COMPLETION_TRACKER.md#reporting) and calculated on the
workbook's *Dashboard*.

Continue normal automated security updates. Low-priority non-applicable cases
need no special deadline.

## 11. Remediation without a full application upgrade

A supported trust-data update, targeted package fix or configuration change
may resolve the exposure while keeping the application version. Prefer the
vendor's remedy; require demonstrated rejection of the compromised chains,
not migration to the newest major version. Rebuild and redeploy containers
where needed, and make sure the change survives restart and scale-out. Don't
restore vulnerable trust silently to fix an outage; raise an exception.

Operational steps are in the [playbooks](playbooks/README.md). Version-aware
detail, vendor procedures and caveats are in
[Remediation options](REMEDIATION_OPTIONS.md).

## 12. Communications, service desk and legal

- **Staff:** no all-staff action is needed on managed devices. Use the service
  desk knowledge article only, unless exploitation is found (decision D-16).
- **Service desk:** triage certificate errors from 27 October, and never
  advise users to bypass warnings
  ([PB-15](playbooks/PB-15-service-desk.md)).
- **Legal and regulatory:** exposure alone is not expected to trigger
  notification. Legal confirms which regulators and privacy authorities apply
  in each jurisdiction where we operate, and the triggers if evidence of
  interception or data exposure is found (decision D-11).
- **Suppliers:** see §6.

## 13. What is not universally required

No new scanner, CBOM, full certificate inventory, bespoke test harness,
exhaustive source audit, per-instance certificate fingerprint or independent
reviewer is required for every submission. Use
[Application validation](APPLICATION_VALIDATION.md) for Route B and central
assurance work, and
[CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md) only where deeper
discovery helps, or for longer-term improvement.

## 14. Distribution brief

**What to send to application, IT and support teams:**
[`playbooks/README.md`](playbooks/README.md). It is self-contained: what has
happened, what to do, what to return and by when, and links to each team's
playbook.

> **To:** named service or platform owner
>
> Eight eMudhra intermediate CA keys were compromised. Browsers are protected;
> software that keeps its own trust list may not be. Please open the team
> guide (`playbooks/README.md`), find your playbook, and return the six-field
> record for each service you own. Work in priority order: P1 items must be
> contained by <date>. State any custom trust configuration or uncertainty.
> Agree remaining dates through the existing change process. Escalate critical
> exposure, outages and blockers immediately to <incident lead / queue>.

No recipients have been identified and no messages have been sent.

## Appendix A: Decision register

All decisions are open. The workbook's *Decisions* sheet holds the same
register, with columns for the decision taken, the rationale and the date.

| ID | Decision | Options | Recommendation | Owner | Forum | Needed by |
|---|---|---|---|---|---|---|
| D-01 | Prioritisation model | (1) v1 four tiers; (2) impact × verifier susceptibility; (3) per-platform lists | (2), as in [Prioritisation](RISK_TIERS.md) | Incident lead | Response steering | 28 Oct |
| D-02 | Browser and desktop closure | (1) treat as fixed; (2) accept with the four-condition proviso and a coverage report; (3) per-device testing | (2), threshold 95% of in-scope devices (§5) | Head of EUC | Response steering | 28 Oct |
| D-03 | Target windows | (1) none, agree per service; (2) indicative P1 72 h contain / 7 d close, P2 14 d, P3 30 d, P4 normal cycle; (3) fixed SLAs | (2); overdue items reported, not penalised | CISO | Risk committee | 28 Oct |
| D-04 | Our certificates under emSign ECC Root CA - G3 | (1) replace external now, internal within 14 days; (2) wait for eMudhra migration support; (3) accept Chrome failures for low-traffic sites | (1); no extension for customer- or partner-facing endpoints | Head of PKI | Incident lead | 27 Oct |
| D-05 | Root removal in our managed stores | (1) remove and handle breakages by time-boxed exception; (2) delay until partners migrate; (3) distrust intermediates only where supported | (1), plus (3) where supported (Windows, Java blocklists). Never restore trust globally | Incident lead | Response steering | 29 Oct |
| D-06 | AAA route on Windows and Java | (1) distrust the specific intermediates and cross-certificates; (2) remove the AAA root; (3) rely on Sectigo revocation | (1); remove AAA only where the vendor prescribes it, after a positive-control test | Head of PKI | Incident lead | 29 Oct |
| D-07 | Inspection-proxy upstream policy | (1) block; (2) warn and allow; (3) re-sign with an untrusted CA | (1), with an exception path for business-critical partners | Network security lead | Incident lead | 28 Oct |
| D-08 | Retrospective hunt scope | (1) none; (2) H1–H6 over available retention, gap recorded; (3) restore archives for the full window | (2) now; (3) only for P1 sources if (2) finds anything | SOC lead | Incident lead | 30 Oct |
| D-09 | Long-tail cut-off and exception policy | (1) no cut-off; (2) cut-off, then exceptions with approved compensating controls; (3) hard deadline with shutdown | (2), cut-off 25 Nov 2026 | CISO | Risk committee | 2 Nov |
| D-10 | Supplier assurance | (1) rely on public statements; (2) formal request to critical suppliers; (3) formal request to all | (2) | Supplier risk lead | Response steering | 2 Nov |
| D-11 | Regulatory and notification stance | (1) no position; (2) no notification for exposure alone, with Legal-defined triggers on evidence of exploitation; (3) precautionary notification | (2); Legal confirms regulators per jurisdiction | General Counsel | Risk committee | 30 Oct |
| D-12 | Reporting unit and cadence | (1) count deployments; (2) count services, executive update twice weekly; (3) daily executive updates | (2); daily only for open P1, blockers and outages | Incident lead | Response steering | 28 Oct |
| D-13 | Internal test service | (1) offline chain bundle only; (2) internal test service run by PKI; (3) temporary egress exceptions | (1) by default; (2) if more than a handful of P1 systems can't be tested | Head of PKI | Incident lead | 30 Oct |
| D-14 | Incident closure criteria | (1) when browsers are fixed; (2) the criteria in §2; (3) when everything is closed | (2) | CISO | Risk committee | 2 Nov |
| D-15 | In-house mobile apps | (1) no action; (2) review apps handling credentials or payments, with minimum-version enforcement if affected; (3) review all apps | (2) | Head of mobile | Response steering | 2 Nov |
| D-16 | Staff communications | (1) all-staff notice; (2) service desk article only; (3) none | (2); all-staff notice only if exploitation is found | Head of communications | Incident lead | 27 Oct |

## References

**Related documents**

- [README](README.md): executive summary, situation and audience map
- [Prioritisation](RISK_TIERS.md), [Completion tracker](COMPLETION_TRACKER.md), [Playbooks](playbooks/README.md)
- Specialist references: [Remediation options](REMEDIATION_OPTIONS.md), [Application validation](APPLICATION_VALIDATION.md), [CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md)
- Workbook: `eMudhra-response-workbook.xlsx` (decisions, tracker, dashboard). Briefing deck: `eMudhra-response-briefing.pptx`

**External sources**

1. Cloudflare: [8 eMudhra certificates are compromised. Here's what you need to know](https://blog.cloudflare.com/emudhra-certificate-compromise/)
2. Incident reference: [Impact, maintained package list and test fixtures](https://emudhra.we-have-a-problem.com/)
3. Google Security Blog: [Protecting Chrome and Android users from compromised eMudhra intermediate CA certificates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
4. Mozilla: [Protecting Firefox users following the compromise of eMudhra intermediate certificates](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/)
5. BleepingComputer: [Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
6. Technobezz: [Google blocks compromised eMudhra keys in Chrome and Android](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android)
7. lilting.ch: [Testing curl, Node.js and uv on macOS against the recovered keys](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
8. iTnews: [Cloudflare warns fixing botched eMudhra certs goes beyond browsers](https://www.itnews.com.au/news/cloudflare-warns-fixing-botched-emudhra-certs-goes-beyond-browsers-629804)
9. certifi on PyPI: [release history](https://pypi.org/project/certifi/#history); webpki-roots on crates.io: [versions](https://crates.io/crates/webpki-roots/versions)
10. Microsoft Security Response Center: [Microsoft's response to compromised eMudhra certificate authority keys](https://www.microsoft.com/en-us/msrc/blog/2026/10/microsoft-response-to-compromised-emudhra-certificate-authority-keys)
11. The Hacker News: [eMudhra to exit global SSL/TLS business](https://thehackernews.com/2026/10/emudhra-to-exit-global-ssltls-business.html)

This is a proposed enterprise workflow. It does not assert that enterprise
systems have been assessed or remediated.
