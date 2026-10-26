# PB-12 Code signing, application control and software distribution

**Owner:** endpoint security and software distribution · SOC for the hunt · **Typical priority:** P1

## Applies if

- You operate **application control** (App Control for Business/WDAC,
  AppLocker, third-party allow-listing) with publisher or certificate rules.
- Or EDR or AV policy treats validly signed binaries more leniently.
- Or you distribute software and drivers, or verify signatures in a pipeline.

## Why this matters

Two of the eight compromised intermediates are **code-signing** CAs, reported
as trusted by Microsoft. With the key, anyone could sign a binary so that
Windows reported a valid signature until the Microsoft block (20 Oct) reached
the device. The keys were weak from generation on **30 April 2025**.

eMudhra states the affected intermediates issued **no end-entity
certificates**. If that is correct, **any signature chaining to them is
presumptively forged**, and a single hit is an incident rather than one more
finding to triage.

## Fix

1. **Windows distrust.** Confirm every Windows endpoint and server has the
   Microsoft block (PB-01 condition 1, PB-02). Where automatic updates are
   off, deploy the two code-signing intermediates (and the TLS ones) to the
   **Untrusted Certificates** store through Group Policy or MDM.
2. **Review application-control rules.**
   - Rules that match on **publisher name only** (for example AppLocker
     publisher rules on the leaf certificate's subject) could be satisfied by
     a forged certificate claiming that publisher, while the chain was trusted.
   - App Control for Business signer rules that **pin the issuing CA** (the
     PCA certificate) are less exposed, unless a rule pins one of the affected
     intermediates. Check for that.
   - Add an explicit **deny** for the affected intermediates if your product
     supports certificate-based deny rules.
3. **EDR and AV.** Check whether any exclusion or reputation rule trusts
   "signed by a trusted publisher" without naming the CA. Tighten it where the
   affected intermediates could qualify.
4. **Software distribution and pipelines.** Signature checks on
   Windows-signed artefacts (installers, drivers, MSIX) rely on the same
   trust. Confirm the verifying hosts have the block.

macOS (Apple-issued Developer ID), Linux package signing (GPG) and mobile app
stores use different trust and are not affected by these intermediates.

## Hunt (with the SOC; see PB-14)

Microsoft Defender for Endpoint, Advanced Hunting. Adapt it to your EDR and
use the exact issuer names from the Security/PKI reference:

```kql
DeviceFileCertificateInfo
| where Issuer has_any ("emSign", "eMudhra")          // narrow with exact intermediate names
| summarize Devices = dcount(DeviceId), FirstSeen = min(Timestamp), LastSeen = max(Timestamp)
          by SHA1, Signer, Issuer, IsTrusted
| order by FirstSeen asc
```

- Many legitimate files are signed under **other**, unaffected emSign CAs.
  Filter to the affected intermediates by exact name and serial or thumbprint
  before raising an incident.
- Advanced Hunting keeps about 30 days. Search longer-retention stores
  (SIEM, data lake, EDR archive) as far back as they go, and **record the gap**
  to 30 April 2025 as a documented limitation (decision D-08).

## Record

- **Route A** for the Windows distrust (shared with PB-01/PB-02).
- Application-control review: rules checked, changes made, deny rules added
  (Route B evidence in the change record).
- Hunt: data sources, window searched, hits and disposition, on the workbook
  **Hunt log** sheet.

## Escalate when

- **Any** binary is found with a signature chaining to an affected
  code-signing intermediate, apart from known research samples. Treat it as a
  suspected compromise.
- Publisher-only allow rules protect high-value systems, and you can't tighten
  them quickly.

Depth: [CBOM_AND_CROSS_SIGNING: Scope beyond HTTPS](../CBOM_AND_CROSS_SIGNING.md#scope-beyond-https).
