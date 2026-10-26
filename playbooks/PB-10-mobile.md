# PB-10 Mobile devices and in-house mobile apps

**Owner:** mobile / MDM team (fleet) · mobile app teams (apps we build) · **Typical priority:** fleet P3–P4 under the proviso; in-house apps handling credentials or payments P1–P2

## Applies if

- You manage Android or iOS devices through MDM or EMM.
- Or you build mobile apps that call our APIs or third-party APIs.

## Managed fleet

| Platform | Vendor protection (reported) | What to confirm |
|---|---|---|
| Android with Google Play services | Equivalent block delivered through Google Play services; a restart triggers it | MDM: devices checked in since 20 Oct; Play services current; one representative device fails the PB-00 test hosts in Chrome |
| Android **without** Google Play services (AOSP builds, some rugged, kiosk or regional devices) | Not covered by the Play services delivery | OEM statement or OS update; until then, treat as long tail (PB-11) |
| iOS / iPadOS | Apple's block, 19 Oct | Devices checked in and on supported versions; one representative device fails the test hosts in Safari |

Apply the same four proviso conditions as PB-01: reachability,
currency, interception and images. Kiosk and shared devices that rarely go
online need a check.

## In-house apps

The OS update doesn't necessarily cover the app. Check how each app decides
trust:

| App trust configuration | Exposure | Action |
|---|---|---|
| Platform default trust, no custom anchors (Android Network Security Config `system` anchors; iOS default ATS) | Follows the OS. Confirm with Google and Apple guidance that the platform verifier carries the block | Representative test on a current device |
| **Custom trust anchors bundled in the app** (`<certificates src="@raw/...">`, a bundled PEM, a custom `TrustManager` or `URLSession` delegate) | Bundled roots don't change with the OS | Ship an app update without the affected root, or with the scoped CAs your backends actually use |
| **Cross-platform frameworks or SDKs with their own TLS stack or CA list** | Same as bundled anchors | Check the framework's CA source; update the SDK |
| **Certificate or public-key pinning** to your own backend's certificate or CA | A forged certificate for a *pinned* host is rejected | Lower risk for pinned hosts; unpinned third-party calls from the app are still exposed |

Search the code for trust-all `X509TrustManager`s,
`onReceivedSslError` → `proceed()`, ATS exceptions
(`NSAllowsArbitraryLoads`), and `URLSession` delegates that accept any
`serverTrust`.

## Verify

Point a debug build at the PB-00 test hosts through the app's own networking
stack. Expect a TLS failure. Then run a normal login and API call.

## Record

- Fleet: a **Route A record per platform** (MDM coverage %, representative
  test, excluded device classes).
- In-house apps: **Route B** per app (trust configuration, test result, store
  release that fixed it).

## Watch out for

- App updates take time to reach users. Track **adoption** of the fixed
  version, and consider enforcing a minimum version for apps handling
  credentials or payments (decision D-15).
- Customer-facing apps on unmanaged devices are outside fleet assurance, but
  inside our duty of care. Record the app trust assessment.

## Escalate when

- An in-house app that handles credentials or payments bundles the affected
  root, or trusts everything.
- A device class without Play services has no OEM fix.

## References

**Related documents**

- [Team guide](README.md)
- [Action plan §5: browser and desktop proviso](../ACTION_PLAN.md#5-phase-1-browser-and-desktop-proviso)
- [Decision D-15 (in-house mobile apps)](../ACTION_PLAN.md#appendix-a-decision-register)

**External sources**

- [Technobezz: Android protection via Google Play services](https://www.technobezz.com/news/google-blocks-compromised-emudhra-keys-chrome-android)
- [Google Security Blog: protecting Chrome and Android users from compromised eMudhra intermediates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
- [BleepingComputer: Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
- [Android: network security configuration](https://developer.android.com/privacy-and-security/security-config)
