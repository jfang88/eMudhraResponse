# PB-13 Our certificates, partner endpoints and availability

**Owner:** PKI / certificate management · service owners of affected endpoints · **Typical priority:** time-critical (Chrome change on **Tuesday 27 October 2026**)

## Applies if

- **We present** certificates issued by eMudhra/emSign: public websites, APIs,
  mail and VPN gateways, internal services using public certificates.
- Or **we call** partner or supplier endpoints whose certificates chain to
  emSign roots. These break when our trust stores remove those roots.

This is an **availability** track, separate from security priority. A
certificate *issued to us* by eMudhra is not compromised by this incident. It
will simply stop being trusted as roots are removed.

## What is changing

| Change | Effect |
|---|---|
| **27 Oct**: Chrome removes emSign ECC Root CA - G3 from the Chrome Root Store | Sites with certificates under that root fail in Chrome. eMudhra's initial estimate is about 30,000 certificates needing migration |
| Mozilla NSS, Ubuntu `ca-certificates`, certifi ≥ 2026.10.18, webpki-roots ≥ 1.0.10 remove emSign ECC Root CA - G3 | Our non-browser clients (after we patch them) fail against partners using that root |
| certifi 2026.10.23 and webpki-roots 1.0.11 also drop other emSign roots (only emSign Root CA - G1 remains in certifi) | A wider set of partner certificates may break on those versions |

## Find

**Certificates we present:**

1. Export from the certificate-lifecycle tool or CMDB all certificates with
   issuer organisation `eMudhra` or `emSign`, and record the root each chains
   to.
2. Search Certificate Transparency for **our domains** (crt.sh or our CT
   monitoring) and filter on issuer `eMudhra`/`emSign`. This catches
   certificates bought outside the central process.
3. Scan our external and internal TLS endpoints, recording the issuer and root
   of the presented chain:

   ```bash
   # One endpoint: show issuer and root of the presented chain
   openssl s_client -connect host.example.com:443 -servername host.example.com -showcerts </dev/null 2>/dev/null \
     | grep -E '^ *[0-9]+ s:|^ *i:'
   ```

**Partner endpoints we call:**

- From TLS-inspection proxy logs, or our own clients' logs, list destinations
  whose upstream chain includes eMudhra/emSign (PB-09 shares the query).
- Ask the owners of P1 integrations to confirm their partners' issuers.

## Fix

- **Our certificates under emSign ECC Root CA - G3**: replace them with a
  certificate from another publicly trusted CA **now**. External sites should
  be done by 27 Oct, or as soon after as possible (decision D-04). eMudhra has
  committed to helping affected subscribers migrate.
- **Other emSign roots**: plan replacement too, given the certifi and
  webpki-roots churn above.
- **Partners**: tell affected partners their certificate will fail in Chrome
  and in updated client libraries, and agree a date. If business-critical
  traffic breaks after we remove the root, raise a **time-boxed exception**
  with a compensating control (for example pin that partner's specific
  certificate in that one client). **Don't restore the root globally.**

## Verify

- New certificate deployed on every node behind the load balancer, with the
  full chain served (re-run the `openssl s_client` check).
- External check from a current Chrome and from a client with an updated
  bundle.

## Record

Use the workbook's **Availability** sheet: endpoint, owner, issuer and root,
replacement CA, replacement date, status. Mark the service's tracker record
with **Availability flag = Yes** until replaced.

## Watch out for

- Certificates on **appliances** (VPN, mail gateways, load balancers) are
  often missed by the lifecycle tool.
- **Pinned** clients (mobile apps, partner integrations pinning our
  certificate or CA) need coordinated updates before we change certificates.
- Service desk will see the user-facing symptoms first. Brief them (PB-15).

## Escalate when

- A customer-facing or partner-facing endpoint under emSign ECC Root CA - G3
  can't be replaced by 27 Oct.
- A critical partner can't change its certificate, and the only workaround
  would restore broad trust.

## References

**Related documents**

- [Team guide](README.md)
- [Action plan §7: availability track](../ACTION_PLAN.md#7-availability-track-tuesday-27-october)
- [Decisions D-04 and D-05](../ACTION_PLAN.md#appendix-a-decision-register)

**External sources**

- [Google Security Blog: protecting Chrome and Android users from compromised eMudhra intermediates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
- [PyPI: certifi release history](https://pypi.org/project/certifi/#history)
- [crates.io: webpki-roots versions](https://crates.io/crates/webpki-roots/versions)
- [Launchpad: Ubuntu ca-certificates bug 2171507](https://bugs.launchpad.net/ubuntu/+source/ca-certificates/+bug/2171507)
- [crt.sh Certificate Transparency search](https://crt.sh/)
