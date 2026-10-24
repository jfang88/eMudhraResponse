# Remediation options — specialist reference

**24 October 2026 | Optional reference for platform and application owners**

Use only the sections relevant to your platform. The [action plan](ACTION_PLAN.md) defines the three acceptable closure routes; this reference does not impose additional evidence on every application. Vendor guidance below was checked on 24 October 2026 and should be rechecked before implementation.

## Remediation without a full application upgrade

**Policy:** require demonstrated rejection of the compromised chains, not migration to the newest application or runtime major version. Prefer a supported trust-data fix or maintenance-branch security update when it covers the application's actual verifier. Keep unrelated lifecycle/security findings separate; incident mitigation does not make unsupported software generally safe.

### Verified delivery options (24 October 2026)

| Platform | Option that can avoid a full application upgrade | Conditions / limitations |
| --- | --- | --- |
| Ubuntu | Targeted updates to ca-certificates and libnss3 rather than upgrading all packages. [R1] | Only covers consumers of those components. Canonical documents an additional Java-store synchronisation issue; preserve locally provisioned CAs when following its workaround. Snap apps have separate update paths. |
| Windows | Microsoft trusted/disallowed Certificate Trust List updates; supported internal distribution for disconnected environments. [R2] | Trust-data delivery, not a universal application installer. Prove effective distrust under the service identity; private Java/PEM/application stores are separate. |
| Azul Zulu | Vendor-provided replacement blocklist file for existing installations, with platform-specific downloads and checksums. [R3] | Follow the exact Zulu/version/platform guidance; replace the correct blocked.certs or blacklisted.certs file, preserve permissions and restart all consuming JVMs. Do not assume it is supported on other JDK vendors. |
| Oracle JDK | Vendor-documented removal of relevant roots from the effective keystore, or enabled revocation checking. [R4] | Oracle's 23 October advisory requires additional action even for listed updated builds. Apply to each installation/custom store as applicable. Root removal can affect other services; revocation requires validation of network reachability and failure behaviour. |
| Python using certifi | Update only the CA-bundle dependency to a compatible fixed release, or configure the HTTP client to use a separately managed bundle where supported. [R5][R6] | No need to assume all application dependencies must change. Certifi does not support editing its packaged bundle in place. Requests supports an explicit verify path or REQUESTS_CA_BUNDLE, but confirm the actual call path honours it. |
| Embedded roots / custom verifiers / vendor appliances | Vendor hotfix, supported maintenance release, or a documented external-store configuration if available. | If the verifier cannot consume external trust/distrust policy, a store-only fix may not exist; a runtime rebuild or vendor update can be necessary. Obtain product-specific confirmation. |
| Containers | Rebuild the existing application release with updated trust data, or deploy a versioned managed trust volume when the application supports it. | Preserve application version where feasible. Changes must survive restart and scale-out; confirm the actual workload reads the changed store. |

These are distinct mechanisms: package update, data-file replacement, trust-list distribution and configuration change. There is no single verified standalone installer that fixes every enterprise verifier.

### Choose the smallest effective change

1. Identify the actual verifier, effective stores and vendor support constraints.
2. Prefer an applicable vendor trust-data/blocklist update or targeted package fix.
3. If unavailable, assess a documented external trust-store override or carefully controlled trust-policy change.
4. If those paths cannot block the affected chains, request a supported hotfix/backport or maintenance release. Escalate an exception if none is available.
5. Record support status and validate both emSign and AAA cross-sign routes, normal business connections and persistence after restart/redeployment.

### Rules for managed trust-store changes

- **Deletion is not distrust:** deleting an intermediate from a local cache/store does not block a peer from supplying it again. Use an enforced distrust mechanism or remove all relevant trust-anchor paths.
- **An additive bundle is insufficient:** for example, NODE_EXTRA_CA_CERTS adds trust; it does not subtract an already trusted built-in CA. Select a documented replacement mechanism for the actual Node version/client where available. [R7]
- **Plain PEM root bundles normally do not express a portable intermediate denylist.** A broad anchor removal may be the available remedy, but it can break legitimate chains. Apply vendor-specific instructions; do not delete roots by a loose subject-name match.
- **Revocation is conditional:** confirm intermediate checks are active, required CRL/OCSP endpoints are reachable and unavailable-status behaviour is acceptable. Merely setting a flag is not completion.
- Version the resulting trust data and record its source, hashes, exact certificate identities and owner. Preserve required private enterprise CAs. Make the change repeatable in images and configuration management so reinstalls do not undo it.
- Retain backups for recovery, but do not silently restore vulnerable trust to resolve an outage. Address legitimate chain issues or use an approved compensating control.
- A custom verifier can ignore all these settings. Validate through the real application client before accepting a store-only remedy.

### Additional completion evidence

Keep relevant implementation details in the existing platform/change record: method, applicability, activation and validation. Hashes are useful for custom trust-data changes, not mandatory for every application using a standard vendor fix. Applications may reference the shared record. Temporary remedies need an owner and review date.

### Remediation references

- [R1: Canonical targeted fixes and Java-store caveat](https://ubuntu.com/blog/emudhra-ca-compromise-fixes-available).
- [R2: Microsoft CTL and disallowed-certificate distribution](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates).
- [R3: Azul downloadable blocklist workaround](https://docs.azul.com/core/emudhra-workaround).
- [R4: Oracle JDK eMudhra guidance, updated 23 October](https://ops.java/emudhra).
- [R5: Certifi bundle and modification policy](https://github.com/certifi/python-certifi).
- [R6: Requests alternate CA-bundle configuration](https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification).
- [R7: Node.js CA-selection configuration](https://nodejs.org/api/cli.html).

These options are a decision framework, not instructions already applied to the enterprise. Recheck the exact vendor advisory before implementation.

