# PB-14 SOC: retrospective hunt and monitoring

**Owner:** SOC / incident response · **Typical priority:** P1 hunts within the first week (decision D-08)

## Applies if

Always. This is a **small, targeted hunt**, not a historical investigation of
every application. It is cheap because of one fact. eMudhra states the affected intermediates issued no end-entity
certificates, so **any certificate chaining to them, outside known research
and test hosts, is presumptively forged**.

## Exposure window

- **Start**: when each intermediate certificate was issued. The keys were
  generated on 30 April 2025; take each issue date from the Security/PKI
  reference.
- **End**: when each client was protected. That was 17–20 Oct 2026 for
  browsers and OS stores, and **later** for long-tail clients still being
  fixed.
- That is up to about 18 months, longer than most log retention. Hunt what you
  hold and **record the gap**.

## Hunts

| # | Data source | Look for | Notes |
|---|---|---|---|
| H1 | **EDR file-signature telemetry** | Binaries signed by certificates issued by the two code-signing intermediates | Query in PB-12. Highest value: endpoint telemetry often has the longest look-back |
| H2 | **TLS-inspection / SWG logs** that record the upstream certificate issuer or chain | Upstream certificates issued by the six TLS intermediates (match the exact issuer DN; SPKI or thumbprint if logged) | Exclude `*.we-have-a-problem.com` (staff testing) |
| H3 | **NDR / Zeek `x509.log`** | Same issuers | **TLS 1.3 encrypts certificates**, so passive sensors only see TLS 1.2 and older handshakes. State this limitation |
| H4 | **Endpoint or app TLS logs** on P1 servers (where an app logs peer certificate details) | Same issuers | Usually sparse; check P1 integrations only |
| H5 | **Certificate Transparency** | Certificates chaining to the intermediates for **our** domains | Google reports none before the block. CT can't show forgeries that were never logged, which is exactly what a targeted attack on non-CT clients would use |
| H6 | **Identity-provider and API logs** | Unexpected JWKS `kid` values, SAML signing-certificate changes, or token-validation anomalies on P1 identity paths (PB-16) | Indirect signal of a forged identity endpoint |

Example Zeek search (adapt to your SIEM):

```text
x509.certificate.issuer contains "emSign"     // then filter to the exact affected intermediate subjects
| exclude server_name ends with ".we-have-a-problem.com"
```

## Ongoing monitoring (until the long tail closes)

- Alert on H1 and H2 matches in real time.
- Alert on long-tail exceptions whose compensating control fails, for example
  a device with restricted egress contacting a new destination.

## Triage a hit

1. Confirm the match is to an **affected** intermediate, not an unaffected
   emSign CA.
2. Exclude known test and research hosts and samples.
3. Treat any remaining hit as a **suspected compromise**: scope the client,
   the destination and the data or credentials involved, and contain it
   through the incident process.
4. Bring in Legal for the notification assessment (decision D-11).

## Record

Workbook **Hunt log** sheet: hunt ID, data source, retention start date,
window searched, query reference, hits, disposition, analyst, date. The
retention gap per source is the documented limitation.

## Escalate when

- Any confirmed hit remains after triage steps 1 and 2.
- A data source needed for a P1 hunt has no retention for the period.

## References

**Related documents**

- [Team guide](README.md)
- [Action plan §8: retrospective hunt](../ACTION_PLAN.md#8-retrospective-hunt)
- [Decisions D-08 and D-11](../ACTION_PLAN.md#appendix-a-decision-register)

**External sources**

- [BleepingComputer: Mozilla, Google, Apple block compromised eMudhra CA certificates](https://bleepingcomputer.com/news/security/mozilla-google-apple-block-compromised-emudhra-ca-certificates)
- [iTnews: Cloudflare warns fixing eMudhra certs goes beyond browsers](https://www.itnews.com.au/news/cloudflare-warns-fixing-botched-emudhra-certs-goes-beyond-browsers-629804)
- [Google Security Blog: protecting Chrome and Android users from compromised eMudhra intermediates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
- [Microsoft Defender: DeviceFileCertificateInfo table](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-devicefilecertificateinfo-table)
- [Zeek: x509.log](https://docs.zeek.org/en/master/logs/x509.html)
- [crt.sh Certificate Transparency search](https://crt.sh/)
