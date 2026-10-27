# Prioritisation: impact and verifier susceptibility

**Version 2 · 26 October 2026 · Proposed enterprise triage model, not a vendor severity rating · Approval: decision D-01 ([Action plan, Appendix A](ACTION_PLAN.md#appendix-a-decision-register))**

## 1. Management principle

Rank each **service or shared component** by what it does with certificates
and what an attacker could do with a forged one. Internet exposure alone does
not decide priority. A public website presenting its own certificate is not
the risk. An internal batch server sending customer data to a SaaS API can be.

Priority comes from two questions answered separately:

1. **Impact.** If this verifier accepted a forged certificate, what could an
   attacker impersonate, read, change or run?
2. **Verifier susceptibility.** Would this verifier, as deployed today,
   actually accept a chain through the affected keys?

For this incident the second question decides almost everything. Browsers and
OS trust stores were protected by their vendors on 17–20 October. Software that
carries its own trust list was not. Ranking by impact alone would put
already-protected desktops ahead of exposed server-side clients.

Keep remediation and evidence scoped to the component. A critical agent on a
host does not impose P1 testing on unrelated applications on the same host.
Use the highest unresolved priority for management escalation.

**How an attacker uses a forged certificate.** To impersonate a server, the
attacker needs to intercept or redirect the victim's connection (DNS,
routing, proxy or local network position). For inbound client-certificate
authentication (mTLS), the attacker can connect directly; exploitation then
depends on the accepted certificate purpose, chain policy and identity
mapping.

## 2. Axis 1: impact

| Impact | Definition | Typical paths |
|---|---|---|
| **H** | A forged certificate could expose credentials or sensitive or regulated data, enable privileged actions or code execution, or affect many downstream systems | Payment and customer-data APIs; identity integrations (OIDC/JWKS, SAML metadata, token endpoints); privileged automation; software-update, build and artifact pipelines; TLS-inspection and egress proxies; internet-facing mTLS; code signing and application control; admin and developer tooling holding cloud credentials or signing keys |
| **M** | Business data or service integrity affected, with limited blast radius | General outbound API clients; internal service-to-service TLS that accepts public roots; internal proxies and application-to-backend TLS; general-purpose desktops |
| **L** | Low consequence, or a demonstrably limited role | Serve-only websites with no relevant outbound validation or client-certificate trust; internal systems restricted to an unaffected private CA with no public-root fallback |

Internal location is not proof of protection. Insiders, compromised hosts and
DNS or routing manipulation can enable impersonation. Raise impact where lower-trust
networks or attacker access make it more likely.

**Records created under version 1 tiers:** Tier 1 → H; Tier 2 and 3 → M (or H
if a promotion condition applied); Tier 4 → L.

## 3. Axis 2: verifier susceptibility

Identify the verifier that actually makes the trust decision: runtime, TLS
library and effective trust store, under the service identity. A host can have
several (OS store, Java store, Python virtual environments, a bundled browser
engine, a proxy).

| Code | Verifier type | Examples | Status |
|---|---|---|---|
| **A** | Already rejects the forged chain: a vendor fix has reached it and the test passes | Chrome, Edge, Firefox, Safari; Windows and Apple trust stores updated by the vendor; Android with current Google Play services; any runtime once its fix is applied and tested (for example a Go binary rebuilt on 1.27.3) | Protected. Confirm coverage; browsers and OS stores count only under the **[browser and desktop proviso](ACTION_PLAN.md#5-phase-1-browser-and-desktop-proviso)** |
| **B** | Uses the OS store; an update exists but isn't yet confirmed on this host | .NET and PowerShell on Windows; curl, PHP, Ruby, Go and Python `ssl` defaults on Linux; apps using the macOS Keychain | Protected once the OS update is confirmed on that host |
| **C** | Own bundle or compiled roots | Java `cacerts`, Python certifi, Node built-in roots, Go and Rust binaries with embedded roots, container images | Exposed until the bundle is updated or the binary or image is rebuilt |
| **D** | Embedded, appliance or vendor-controlled; no fix yet | Appliances, agents, packaged products with embedded runtimes, OT/IoT, SaaS | Exposed until the vendor fixes it; compensating control needed meanwhile |
| **Unknown** | Not yet assessed | — | Assess at the priority shown in the matrix. Unknown is not low |

**The TLS-inspection proxy is the verifier for every client behind it.**
Clients, browsers included, see only the proxy's re-signed certificate, so
vendor browser blocks don't apply to inspected traffic. Classify inspected
paths by the proxy's susceptibility, and treat the proxy itself as H.

**Where the Sectigo AAA route still matters.** The affected keys also have
AAA Certificate Services cross-certificates. Current Mozilla-derived bundles
(for example certifi and Ubuntu's `ca-certificates`) no longer trust AAA for
TLS. Windows, Java and older or vendor-modified bundles may still trust it.
Test both routes there. See
[CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md#sectigo--aaa-relevant-but-not-a-sectigo-root-key-compromise).

**"Vendor fix" means tested, not announced.** A vendor statement says a fix
exists. It doesn't show the fix reached this host, covers both routes, or is
used by this client. Record A only when the [PB-00](playbooks/PB-00-test-and-evidence.md)
test passes through the client the service actually uses.

### How each platform reaches A

No platform is protected by default, and each has its own gaps. Dates are as
at 27 October 2026; see the [README](README.md#situation-as-at-27-october-2026)
for confidence labels.

| Platform | Reaches A when | Stays B, C or D if |
|---|---|---|
| **Windows** (.NET Framework, .NET on Windows, PowerShell 5.1, WinHTTP/SChannel apps) | The Disallowed list update (20 Oct) and the out-of-band trust-list update (26 Oct) are on the host, and both the emSign and AAA routes test clean under the service identity [5] | Automatic root updates are off and the out-of-band update isn't installed from the Update Catalog; code overrides validation (`ServerCertificateCustomValidationCallback`, `CustomRootTrust`); the app is .NET on Linux (OpenSSL store). .NET doesn't check revocation by default, so Sectigo's revocation of the AAA cross-signs doesn't help on its own |
| **Apple** (Safari; apps using the system trust store) | Apple's block (19 Oct) is on the device | The tool reads `/etc/ssl/cert.pem` (the built-in `curl` still accepted the chain after Apple's block); the app bundles its own roots |
| **Browsers** (Chrome, Edge, Firefox) | CRLSet or OneCRL received (17–20 Oct); Chrome and Edge root-store removals on 27 Oct | Traffic is TLS-inspected; component updates are blocked; the app is Electron (no CRLSet) |
| **Android** | Google Play services current | The device has no Play services; the app bundles its own trust anchors |
| **Linux distributions** | `ca-certificates` updated (Ubuntu 24.04: `20260816ubuntu0.24.04.1`) and services restarted | The Java store wasn't regenerated; container images weren't rebuilt; the distribution fix is still pending |
| **Go** | Binary rebuilt on Go 1.27.3 or 1.26.10 | Not rebuilt, or built on Go 1.25 or older |
| **Python** | certifi 2026.10.18 or later deployed and workers restarted | An older certifi is pinned or vendored into a package |
| **Node.js** | Running a security release published on 20–21 Oct (22.23.4, 24.22.1, 26.12.1; confirm in the Node.js release notes), or using the patched OS store via `--use-system-ca` | Older release with built-in roots; only `NODE_EXTRA_CA_CERTS` used (it adds trust, never removes it); Electron apps awaiting their vendor |
| **Java** | Vendor remedy applied (Oracle steps; Azul blocklist), JVMs restarted, and both routes test clean | Another vendor's blocklist was copied in; a custom `TrustManager` ignores the store |
| **Rust** | Rebuilt with webpki-roots 1.0.10 or later, or using the patched OS store | Not rebuilt |
| **Appliances, agents and SaaS** | The vendor confirms a fix for the deployed version and, where possible, a test passes | No vendor statement; the statement names no version |

## 4. Priority matrix

| Impact ↓ / Susceptibility → | A | B | C | D | Unknown |
|---|---|---|---|---|---|
| **H** | P3: confirm coverage | **P1** | **P1** | **P1** + exception and compensating control | **P1 to assess** |
| **M** | P4: confirm coverage | P2 | P2 | P2 + vendor tracking | P2 to assess |
| **L** | P4 | P3 | P3 | P3 + vendor tracking | P3 to assess |

**Where common systems land** (one typical example per cell; the briefing
deck shows the same grid):

| Impact ↓ / Susceptibility → | A: vendor already blocks it | B: OS store, patch unconfirmed | C: own trust list | D: vendor-controlled, no fix | Unknown |
|---|---|---|---|---|---|
| **H** | P3: Go payment service rebuilt on Go 1.27.3, test passed | P1: Windows API validating JWTs; Microsoft updates not yet confirmed | P1: batch job sending customer data to SaaS (Java) | P1: inspection proxy awaiting a vendor fix, with exception | P1 to assess: payment API, runtime not yet identified |
| **M** | P4: managed browser, not TLS-inspected | P2: internal service using the Linux OS store | P2: internal service using certifi or Node's built-in roots | P2: appliance cloud connector, vendor tracked | P2 to assess: internal app not yet assessed |
| **L** | P4: iPad kiosk on current iOS | P3: low-impact script using the system curl | P3: low-impact tool with its own bundle | P3: printer firmware update check, vendor tracked | P3 to assess: low-impact system not yet assessed |

Systems move across the grid as work is done. Confirming an OS update moves
a system from B to A. For example, the Windows API above moves to P3 once the
Microsoft updates are confirmed and both routes test clean. Updating or
rebuilding its bundle moves a system from C to A or B. Systems that are not applicable at all, such as a serve-only website,
close through Route C and sit outside the grid. Section 6 has more worked
cases.

**Indicative target windows** (from discovery or assignment; approval is
decision D-03):

| Priority | Window |
|---|---|
| P1 | Contain within 72 hours; close or except within 7 days |
| P2 | Close or except within 14 days |
| P3 | Close within 30 days |
| P4 | Confirm in the normal cycle |

Suspected exploitation or a critical outage triggers the incident process
immediately, whatever the priority. Dates agreed through change control may
differ from the window; record the reason. Reassigned work keeps its original
date.

## 5. Fix first: paths that turn a forged certificate into something worse

Treat these as **P1** wherever the verifier is not yet confirmed (B, C, D or
Unknown):

| Path | Why | Playbook |
|---|---|---|
| TLS-inspection and egress proxies | Decide trust for every client behind them | [PB-09](playbooks/PB-09-proxies-gateways-and-tls-inspection.md) |
| Identity integrations: OIDC discovery/JWKS, SAML metadata, OAuth token endpoints | A forged JWKS endpoint lets an attacker mint tokens your API accepts; token calls carry secrets | [PB-16](playbooks/PB-16-identity-and-mtls.md) |
| Build pipelines and artifact proxies (Artifactory, Nexus) | A forged certificate becomes code execution in every build | [PB-08](playbooks/PB-08-containers-kubernetes-and-cicd.md) |
| Payment and customer-data APIs | Sensitive data over outbound calls from Java, Python, Node and Go services | [PB-04](playbooks/PB-04-java.md) to [PB-07](playbooks/PB-07-go-rust-php-ruby.md) |
| Inbound mTLS trusting public roots | No interception needed: the attacker connects directly | [PB-16](playbooks/PB-16-identity-and-mtls.md) |
| Code signing and application control | Publisher-name rules could accept forged signatures while Windows trusted the chain | [PB-12](playbooks/PB-12-code-signing-and-application-control.md) |
| Admin and developer workstations | CLI tools and SDKs with cloud credentials or signing keys sit outside the browser proviso | [PB-01](playbooks/PB-01-browsers-and-desktop-os.md) |

## 6. Classify these common cases

| Scenario | Impact | Susceptibility | Priority and treatment |
|---|---|---|---|
| Managed browser, ordinary internet use, not TLS-inspected | M | A | **P4**. Closes under the browser and desktop proviso |
| Managed browser behind a TLS-inspection proxy | M | Proxy's | Inherits the proxy's priority; fix the proxy (PB-09) |
| Public static website; unaffected server certificate; no outbound calls or mTLS | L | — | Route C for the serving process after evidence review. Assess host agents, updates and the deployment pipeline separately |
| Public website whose own certificate chains to emSign ECC Root CA - G3 | L (security) | — | **Availability flag = Y**. Replace the certificate before or soon after 27 Oct ([PB-13](playbooks/PB-13-our-certificates-and-availability.md)) |
| Public website calling an external payment or identity API | H | Usually C | **P1** for those outbound paths, whoever issued the website's certificate |
| Internal batch server sending customer data to a SaaS API | H | C (e.g. Java or Python bundle) | **P1**. No inbound exposure doesn't reduce outbound trust risk |
| API validating JWTs from an external IdP | H | B or C | **P1**. Fix the runtime the identity library uses to fetch JWKS |
| Self-hosted artifact proxy fetching from public registries | H | C (usually Java) | **P1**. Chokepoint for every build |
| Internal low-impact service using a patched OS store and accepting public roots | M | B | **P2** until the OS update is confirmed, then close via the platform record |
| Internal low-impact service on a bundled or compiled store | M | C | **P2** |
| Internal service trusting only a dedicated unaffected private CA | L | — | Route C after verifying no public-root fallback, alternate verifier or external dependency |
| Developer or admin workstation using CLI tools with privileged cloud credentials | H | C | **P1** for those tools; the browser on the same device is P4 |
| Reverse proxy serving users and opening TLS to backends | Per leg | Per leg | Classify the front and back legs separately. The server certificate says nothing about backend validation |
| Container with no listening port but outbound external API calls | By data | Usually C | Image packaging doesn't set impact; the bundle inside it sets susceptibility |
| Endpoint estate with publisher-rule application control | H | Windows distrust status | **P1** for the policy owner (PB-12) |
| Vendor appliance making outbound update or cloud calls, no fix yet | By function | D | Exception with compensating control; track the vendor ETA (PB-11) |

## 7. Flags that sit beside priority

These are tracked in the same record but are **not** priorities:

- **Availability flag.** Set when the service presents, or depends on,
  certificates chaining to emSign roots that are being removed (Chrome on 27
  October; Mozilla-derived bundles). An outage can be urgent even when
  confidentiality exposure is low. See the
  [availability track](ACTION_PLAN.md#7-availability-track-tuesday-27-october).
- **Code-signing relevance.** Set when the component verifies Windows code
  signatures or enforces publisher-based application control. HTTPS tests
  don't cover this purpose.

## 8. Proportionate evidence for non-applicability

A short owner statement supported by existing configuration or architecture
evidence is enough for Route C. State the assessed role: serve-only with no
relevant outbound or client-certificate validation, or private-CA-only with no
public-trust fallback. "We don't buy eMudhra certificates" is not a reason.
Don't require a new exhaustive dependency map to prove a negative.

Refer shared host agents, update services, sidecars and deployment tooling to
their accountable platform owners rather than duplicating their assessment. If
a dependency or fallback is uncertain, keep that part open. Uncertainty is not
evidence of non-applicability.

## 9. Priority, exposure and status are different fields

Record impact, susceptibility, priority and status separately in the
[completion tracker](COMPLETION_TRACKER.md). A protected browser is P4 with
status *Closed — A*. It doesn't need emergency remediation because it reaches
the internet. Unknown is not low. "Certificate found" is not proof of
exploitability, and a clean package scan is not proof of protection.

This incident's priorities don't replace general vulnerability priorities for
internet-facing systems.

## References

**Related documents**

- [Action plan](ACTION_PLAN.md): browser and desktop proviso, long tail, availability track, decision register
- [Completion tracker](COMPLETION_TRACKER.md): fields for impact, susceptibility and priority
- [Playbooks](playbooks/README.md): per-team and per-stack remediation
- [CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md): the AAA cross-sign route

**External sources**

1. Mozilla: [Protecting Firefox users following the compromise of eMudhra intermediate certificates](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/)
2. Cloudflare: [8 eMudhra certificates are compromised. Here's what you need to know](https://blog.cloudflare.com/emudhra-certificate-compromise/)
3. Google Security Blog: [Protecting Chrome and Android users from compromised eMudhra intermediate CA certificates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
4. oss-security: [AAA Certificate Services trust bit discussion, 21 Oct 2026](https://openwall.com/lists/oss-security/2026/10/21/4)
5. Microsoft Security Response Center: [Microsoft's response to compromised eMudhra certificate authority keys](https://www.microsoft.com/en-us/msrc/blog/2026/10/microsoft-response-to-compromised-emudhra-certificate-authority-keys)
6. Node.js: [release index (security releases 20–21 Oct)](https://nodejs.org/dist/index.json); Go: [release history](https://go.dev/doc/devel/release); PyPI: [certifi release history](https://pypi.org/project/certifi/#history); crates.io: [webpki-roots versions](https://crates.io/crates/webpki-roots/versions)
7. lilting.ch: [macOS curl, Node.js and uv tests](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)

Neither Mozilla, Cloudflare nor Google assigns these enterprise priorities.
The model is an enterprise assessment derived from the authentication risks
they describe.
