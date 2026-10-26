# PB-11 Vendor products, appliances, agents and suppliers (the long tail)

**Owner:** the system or service owner, supported by supplier management · **Typical priority:** by impact; susceptibility usually D (vendor-controlled)

## Applies if

The verifier is **not under our direct control**:

- Network and security appliances, storage, backup, hypervisor management.
- Endpoint, monitoring, backup, management, VPN/ZTNA and EDR **agents**.
- Packaged software with an embedded runtime (Java, Node, Python, OpenSSL).
- OT, IoT and facilities: building management, cameras, printers, UPS and
  badge systems with cloud management or update services.
- SaaS and suppliers that process our data, or call our APIs and webhooks.

## Find them

1. **Egress telemetry first.** List internal sources making outbound TLS
   connections (proxy, firewall, NAT and cloud egress logs, last 30 days). Map
   each source to an owner. Anything that isn't a standard platform or a known
   app is a long-tail candidate.
2. **Direct-to-internet** TLS that bypasses the proxy is a priority finding.
3. Add the CMDB, software inventory and the supplier register for critical
   suppliers.
4. Enter each item on the workbook's **Long tail** sheet.

## Fix

1. **Ask the vendor**, using the template below. Record the answer, reference
   and date.
2. **Apply the vendor's fix**: firmware, hotfix, trust-store update or
   configuration.
3. **No fix yet?** Apply an approved **compensating control** and raise an
   exception with a review date tied to the vendor's ETA:
   - Restrict the device's egress to the specific destinations it needs.
   - Route its traffic through the remediated TLS-inspection proxy, if
     compatible (PB-09).
   - Segment the device, or place it behind a gateway that terminates and
     re-validates TLS.
   - For a critical integration, pin the expected CA or key where the product
     allows.
4. **Suppliers and SaaS**: obtain written confirmation for **critical
   suppliers** (payments, identity, SaaS with regulated data, managed
   security providers). For others, rely on their published statements.

## Vendor and supplier request template

> **Subject:** eMudhra intermediate CA key compromise: confirmation request for
> <product / service>
>
> On 17 October 2026, eight eMudhra intermediate CA keys were reported as
> compromised (six TLS and two code-signing). Major browsers and operating
> systems blocked them between 17 and 20 October. Software that uses its own
> trust store may still accept certificates forged with these keys, including
> through the Sectigo "AAA Certificate Services" cross-signs.
>
> For <product, version> / <service> as used by <our organisation>, please
> confirm:
>
> 1. Whether the product or service validates public TLS certificates
>    (outbound connections, update checks, client-certificate authentication)
>    using its own trust store or runtime.
> 2. Whether it rejects chains through the affected intermediates by both the
>    emSign and the AAA route, and from which version, release or date.
> 3. For code-signing: whether any of your signed components chain to the
>    affected code-signing intermediates.
> 4. For services: whether you found any evidence of misuse affecting our data
>    or tenant.
> 5. Any action required from us, such as an upgrade, configuration change or
>    restart.
>
> Please reply by <date>. Reference: <incident ID>.

## Verify

- Where you can test (non-production instance, test VLAN): use the PB-00 test
  hosts through the product's own feature that makes outbound calls.
- Otherwise accept a **product- and version-specific** vendor statement,
  recording applicability and limitations (pack rule). A generic "we are not
  affected" without a version is not enough for P1.

## Record

Long-tail record: vendor status (Fixed / Pending / Not affected / No
response), reference, interim control, exception ID, target date.

## Watch out for

- "We don't use eMudhra certificates" answers the wrong question. Ask what the
  product **trusts**.
- Agents run on many hosts: one vendor fix needs a **coverage** report.
- Vendors fixing by **root removal** can break your legitimate emSign-chained
  partners (PB-13).

## Escalate when

- A P1 item's vendor doesn't respond within the agreed time, or says there
  will be no fix.
- A critical supplier reports possible misuse affecting our data. Call the
  SOC (PB-14) and Legal (decision D-11).
