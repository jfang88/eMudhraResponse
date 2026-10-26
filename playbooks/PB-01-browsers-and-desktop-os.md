# PB-01 Browsers and desktop operating systems

**Owner:** Desktop / end-user computing · **Typical priority:** P3–P4 (vendor-protected), subject to the proviso below

## Applies if

You manage Windows, macOS, ChromeOS or Linux desktops, VDI images, kiosks, or
the browsers on them.

## What the vendors already did

| Platform | Mechanism | Date (2026) | User action needed |
|---|---|---|---|
| Chrome (all desktop OS) | CRLSet block of the intermediates' keys | 17 Oct | None (component update) |
| Chrome | emSign ECC Root CA - G3 removed from the Chrome Root Store | **27 Oct** | None; **legitimate sites under that root will fail** |
| Firefox (incl. ESR) | OneCRL block via Remote Settings | 17 Oct | None |
| Safari / Apple OS | Apple block | 19 Oct | None reported; see the macOS CLI note below |
| Windows (and Edge) | Microsoft block | 20 Oct | None where automatic root and disallowed-certificate updates are on |

Dates come from vendor statements as reported by the press; confirm them on the
workbook **Facts** sheet.

## Fix: usually nothing to install. Confirm the proviso.

Record a configuration as **Validated** only when all four conditions hold.

**1. Reachability.** Devices must be able to fetch vendor updates.

- Chrome: egress to Google's component update service is allowed and no policy
  disables component updates. On a sample device, open `chrome://components`
  and check that **CRLSet** has a recent version.
- Firefox: egress to Mozilla Remote Settings
  (`firefox.settings.services.mozilla.com`) is allowed.
- Windows: automatic root and disallowed-certificate updates are on.

  ```powershell
  # Read-only. A value of 1 for DisableRootAutoUpdate, or 0 for
  # EnableDisallowedCertAutoUpdate, means Windows will not fetch the block itself.
  Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\SystemCertificates\AuthRoot' -ErrorAction SilentlyContinue |
    Select-Object DisableRootAutoUpdate, EnableDisallowedCertAutoUpdate
  ```

  If updates are off by design, for example in a hardened or disconnected
  estate, distribute the disallowed certificates yourself: through the Group
  Policy **Untrusted Certificates** store, using the eight certificates
  supplied by Security/PKI, or through Microsoft's internal CTL distribution
  (`certutil -syncWithWU`). See
  [Microsoft's guidance](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates).

**2. Currency.** Supported browser and OS versions. Use your device-management
reports for the version spread and for devices that haven't checked in since
20 October.

**3. No interception.** If a TLS-inspection proxy decrypts browsing, the
**proxy** decides trust and the browser's protection does not apply to that
traffic. Get the proxy's status from the network team (PB-09) and record it
against each configuration.

**4. Images.** Refresh non-persistent VDI golden images, kiosk images and
offline builds, or test them separately. Devices that were offline for the
whole period need a check-in before they count.

## Verify

Run one representative test per **managed configuration**, not per user. For
example: Windows + Chrome, Windows + Edge, Windows + Firefox ESR, macOS +
Safari, macOS + Chrome, each VDI image, and ChromeOS if used.

- Open each test host from [PB-00](PB-00-test-and-evidence.md) on the managed
  network **and** off-network or over VPN. Expect a certificate error page,
  never the site.
- Open the positive control. It must load.
- Windows, read-only check of the machine's distrusted store:

  ```powershell
  Get-ChildItem Cert:\LocalMachine\Disallowed |
    Where-Object { $_.Subject -match 'emSign|eMudhra' } |
    Select-Object Subject, Thumbprint
  ```

  Compare thumbprints with the Security/PKI reference. If they are missing,
  check condition 1 and rely on the behavioural test.

## The desktop software the proviso does NOT cover

Hand these to their owners. They are part of the long tail.

| Software on the desktop | Why it isn't covered | Go to |
|---|---|---|
| macOS built-in `/usr/bin/curl` and other CLI tools | Uses its own CA file; was shown to accept the chain after Apple's block | PB-03 |
| Java runtimes (desktop apps, IDEs, JNLP) | Own `cacerts` | PB-04 |
| Python, Node and Go developer tools, package managers | Own bundles or compiled roots | PB-05 to PB-07 |
| Electron apps (Slack, VS Code, and others) | No CRLSet; Node-side traffic uses bundled roots | PB-06 |
| VPN, ZTNA, backup, management and EDR agents | Vendor-controlled verifiers | PB-11 |

Find them with your software inventory (Intune, SCCM, Jamf). **Developer and
administrator workstations** that hold privileged cloud credentials or signing
keys are **P1** for those tools.

## Record

- One **Route A assurance record per managed configuration**: mechanism,
  representative test result, the four proviso checks, device coverage %, and
  exclusions such as unmanaged or long-offline devices.
- Fill in the workbook's **Browser & desktop** sheet. It computes "proviso met".
- Tracker: Impact M, Susceptibility A, Priority P4; status *Closed — A* once the
  proviso is met.

## Watch out for

- **27 October is an availability change**, not a new gap. Brief the service
  desk ([PB-15](PB-15-service-desk.md)).
- Don't treat Chrome and Edge as identical: they use different root stores.
  Test both.
- **BYOD and unmanaged devices** are outside enterprise assurance. Say so in the
  record rather than claiming coverage.
- Don't ask every user to test. One test per configuration plus the coverage
  report is enough.

## Escalate when

- A test host loads in a managed browser.
- Policy or egress blocks vendor updates and can't be changed within 72 hours.
- More than a small share of devices haven't checked in since 20 October
  (threshold set by the incident lead).

## References

**Related documents**

- [Team guide](README.md)
- [Action plan §5: browser and desktop proviso](../ACTION_PLAN.md#5-phase-1-browser-and-desktop-proviso)
- [Prioritisation](../RISK_TIERS.md)
- [PB-09 (TLS inspection)](PB-09-proxies-gateways-and-tls-inspection.md)

**External sources**

- [Google Security Blog: protecting Chrome and Android users from compromised eMudhra intermediates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
- [Mozilla: protecting Firefox users following the eMudhra compromise](https://blog.mozilla.org/security/2026/10/20/protecting-firefox-users-following-the-compromise-of-emudhra-intermediate-certificates/)
- [BleepingComputer: Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
- [Free Press Journal: vendor block dates and code-signing intermediates](https://www.freepressjournal.in/tech/why-did-apple-google-microsoft-mozilla-block-certificates-from-indian-firm-emudhra)
- [Microsoft: configure trusted roots and disallowed certificates](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates)
- [lilting.ch: macOS curl, Node.js and uv tests against the recovered keys](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
