# PB-15 Service desk knowledge article

**Owner:** service desk lead, with Security/PKI · **Use from:** now, and especially from **Tuesday 27 October 2026**

## Summary for agents

A certificate authority called eMudhra had eight of its signing keys exposed.
Browser and operating-system vendors **already blocked them automatically**
between 17 and 20 October. **Users don't need to do anything** on managed
devices except keep them switched on, connected and up to date.

From 27 October, Chrome stops trusting one of eMudhra's root certificates.
Some **legitimate** websites that use eMudhra certificates will show a
certificate error in Chrome until those sites change certificate. This is
expected and isn't caused by our devices.

## Common questions

| User asks | Answer |
|---|---|
| "Is my computer or phone safe? Do I need to do anything?" | No action needed on managed devices. Keep the device on, connected and updated, and restart your browser if it has been open for days. |
| "Should I change my password?" | Not because of this. If Security asks you to, they'll contact you directly. |
| "A website shows a certificate error." | Follow the triage steps below. **Never** advise clicking through, installing certificates, or switching browsers to get past it. |
| "My personal phone or laptop?" | Keep it updated. Browsers and phone operating systems have been updated by their vendors. |

## Triage: website or application certificate errors

1. Record the **URL**, browser and version, device, network (office, home,
   VPN), time, and the exact error text. Chrome typically shows
   `NET::ERR_CERT_AUTHORITY_INVALID` or `NET::ERR_CERT_REVOKED`.
2. Ask the user to click the padlock or error details and read the **issuer**.
   Or check it yourself.
3. **Issuer contains "eMudhra" or "emSign"**:
   - **Our own site or app** → P2 ticket to PKI / certificate management
     (PB-13).
   - **Supplier or partner site the business needs** → ticket to the supplier
     owner, who contacts the supplier (PB-13). Business-critical and blocking?
     Escalate to the incident lead.
   - **Other public site** → explain it's the site's certificate problem; no
     ticket needed.
4. **Issuer is our corporate inspection CA** and the error mentions the
   upstream site → network team (PB-09).
5. **An application, not a browser**, reports a certificate or TLS error
   after a recent update → ticket to the application owner, quoting
   "eMudhra response". Don't roll back trust-store updates.

## Never

- Tell users to bypass certificate warnings, import certificates, or turn off
  security software.
- Restore or re-add removed certificates on a device to "fix" a site.

## Escalate immediately to Security

- A user reports a **known internal or major site** (identity provider, bank,
  payroll, our VPN) showing a certificate from an unexpected issuer, or a
  warning they clicked through.
- Several users report the same unexpected certificate on the same site.
