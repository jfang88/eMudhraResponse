# Remediation options — specialist reference

**Version 2 · 26 October 2026 · Specialist reference for platform and application owners**

**Start with your team's [playbook](playbooks/README.md).** This reference gives version-aware detail behind the playbooks. Use only the sections relevant to your platform. The [action plan](ACTION_PLAN.md) defines the three closure routes, and [prioritisation](RISK_TIERS.md) sets the order of work. This reference adds no evidence requirements for every application. Vendor guidance was checked on 24 October 2026 and updated on 26 October; recheck it before implementation.

## Remediation without a full application upgrade

**Policy:** require demonstrated rejection of the compromised chains, not migration to the newest application or runtime major version. Prefer a supported trust-data fix or maintenance-branch security update when it covers the application's actual verifier. Keep unrelated lifecycle/security findings separate; incident mitigation does not make unsupported software generally safe.

### Verified delivery options (updated 26 October 2026)

| Platform | Option that can avoid a full application upgrade | Conditions / limitations |
| --- | --- | --- |
| Ubuntu | Targeted updates to ca-certificates and libnss3 rather than upgrading all packages. On 24.04 LTS, `ca-certificates 20260816ubuntu0.24.04.1` (20 Oct) blocklists emSign ECC Root CA - G3. [R1][R11] | Only covers consumers of those components. Canonical documents an additional Java-store synchronisation issue; preserve locally provisioned CAs when following its workaround. Snap apps have separate update paths. |
| Windows | Microsoft's Disallowed Certificate Trust List update (20 Oct) blocks the affected CAs; an out-of-band update (26 Oct) removes seven related certificates, including an AAA Certificate Services cross-sign, from the Windows trust list. Disconnected machines install it from the Microsoft Update Catalog. Supported internal CTL distribution is available for disconnected environments. [R2][R15] | Trust-data delivery, not a universal application installer. Prove effective distrust under the service identity, on both the emSign and AAA routes: .NET doesn't check revocation by default, so Sectigo's revocation of the cross-signs doesn't protect it. Private Java/PEM/application stores are separate. KB numbers for the out-of-band update are reported by third parties; confirm them in the Update Catalog. |
| Azul Zulu | Vendor-provided replacement blocklist file for existing installations, with platform-specific downloads and checksums. [R3] | Follow the exact Zulu/version/platform guidance; replace the correct blocked.certs or blacklisted.certs file, preserve permissions and restart all consuming JVMs. Do not assume it is supported on other JDK vendors. |
| Oracle JDK | Vendor-documented removal of relevant roots from the effective keystore, or enabled revocation checking. [R4] | Oracle's 23 October advisory requires additional action even for listed updated builds. Apply to each installation/custom store as applicable. Root removal can affect other services; revocation requires validation of network reachability and failure behaviour. |
| Python using certifi | Update only the CA-bundle dependency to **certifi 2026.10.18 or later**, or configure the HTTP client to use a separately managed bundle where supported. [R5][R6][R10] | Checked 26 Oct: 2026.7.22 still contains emSign ECC Root CA - G3; 2026.10.18, .19 and .23 do not. 2026.10.23 also drops emSign Root CA - C1 and emSign ECC Root CA - C3, which affects availability (see the [action plan](ACTION_PLAN.md#7-availability-track-tuesday-27-october)). No need to assume all application dependencies must change. Certifi does not support editing its packaged bundle in place. Requests supports an explicit verify path or REQUESTS_CA_BUNDLE, but confirm the actual call path honours it. |
| Go | Rebuild with **Go 1.27.3 or 1.26.10** or later (released 19 Oct), which reject chains containing any of the eight keys. On Linux, updating the system bundle also removes the root for binaries using system roots. [R8][S6] | A binary is protected only once rebuilt. Go 1.25 and older receive no fix. Embedded root pools and scratch images need a rebuild. |
| Rust webpki-roots | Update the crate to **1.0.10 or later** and rebuild. [R9] | Checked 26 Oct: 1.0.9 still contains emSign ECC Root CA - G3. 1.0.11 drops further emSign roots. Crates using the OS store follow the OS fix. |
| Node.js and Bun | Upgrade within your major line to a Node.js security release published on 20–21 Oct (22.23.4, 24.22.1, 26.12.1), which third-party advisories list as the fix; or use the patched OS store (`--use-system-ca` where the build lists it, otherwise `--use-openssl-ca` on Linux); or set an explicit `ca` on the client the application uses. [R7][R12][R16] | Release existence verified on nodejs.org; confirm the fix in the Node.js security release notes. Node 22.22.0's built-in roots still included emSign ECC Root CA - G3 on 26 Oct. Bun status unconfirmed. `NODE_EXTRA_CA_CERTS` only adds trust. |
| macOS command-line tools | Use Homebrew curl/OpenSSL with updated `ca-certificates`, or pass an approved bundle explicitly, until Apple updates `/etc/ssl/cert.pem`. [R13] | The built-in `/usr/bin/curl` accepted the affected chain after Apple's block. Safari and the Keychain are covered by Apple's fix. |
| Embedded roots / custom verifiers / vendor appliances | Vendor hotfix, supported maintenance release, or a documented external-store configuration if available. | If the verifier cannot consume external trust/distrust policy, a store-only fix may not exist; a runtime rebuild or vendor update can be necessary. Obtain product-specific confirmation. |
| Containers | Rebuild the existing application release with updated trust data, or deploy a versioned managed trust volume when the application supports it. | Preserve application version where feasible. Changes must survive restart and scale-out; confirm the actual workload reads the changed store. |

These are distinct mechanisms: package update, data-file replacement, trust-list distribution, rebuild and configuration change. There is no single verified standalone installer that fixes every enterprise verifier.

**Root removal affects availability.** Most Mozilla-derived fixes remove emSign ECC Root CA - G3 rather than blocking the keys. Legitimate partner certificates under that root then fail for the updated client. Check the availability list ([PB-13](playbooks/PB-13-our-certificates-and-availability.md)) before rolling out, and handle breakages by time-boxed exception, never by restoring the root globally.

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


## Practical recipes for older application stacks

These recipes help central platform owners build reusable assurance patterns. **They are not a request for every application team to run every command.** Use the installed vendor/build's documentation; a feature's historical availability does not mean that release is still supported. An incident-specific trust fix does not resolve unrelated vulnerabilities in an old runtime.

The examples are implementation templates, not executed enterprise changes. Paths such as /opt/company/trust and /path/to/runtime are placeholders. Retain the same application version where possible; distinguish a vendor-issued incident remedy from a documented configuration mechanism that the enterprise must validate.

Quick navigation: [Java](#java-jdk-8-and-later-plus-older-installations) · [Node.js](#nodejs-older-bundled-roots-and-custom-agents) · [Python](#python-keep-the-interpreter-change-the-correct-bundle) · [Windows/.NET](#windows-and-net) · [Linux/curl](#linux-curl-and-openssl-consumers) · [Other stacks](#other-stacks-scope-the-workaround-before-promising-it).

### Shared preparation: produce the right trust data

PKI/platform teams should provide a versioned, integrity-checked replacement bundle or vendor blocklist and document its supported consumers. Prefer applicable vendor trust-data packages. If constructing a restricted bundle, start from the application's approved roots, preserve required private enterprise CAs and remove the exact disallowed anchor certificates identified by vendor guidance and fingerprint.

A plain PEM bundle is a set of trusted CAs, not a general denylist. Do not put compromised intermediates into it to “block” them: that can add trust. Removing only cached intermediates is ineffective when peers supply them again. Account for emSign and AAA alternatives, directly trusted intermediates and any secondary CA directories. Prefer precise vendor distrust mechanisms where available; broader root removal can break unrelated services.

Deploy the new file separately from the live file, inspect it, apply controlled file permissions and switch the application configuration. Restart/reload consumers as appropriate. Keep recovery copies, but treat restoring vulnerable trust as a risk decision. Include the change in image/build/configuration sources so upgrades, rollbacks or scale-out do not undo it.

### Java: JDK 8 and later, plus older installations

**Discover.** Locate the Java executable used by the service, not just the shell's java. Inspect the service wrapper, application-server configuration and startup properties. Useful read-only checks:

```bash
/path/to/runtime/bin/java -version
/path/to/runtime/bin/java -XshowSettings:properties -version
/path/to/runtime/bin/keytool -list -v -keystore /path/to/effective/truststore
```

Use the actual store password interactively; do not paste passwords into commands or logs. Limit output sharing to relevant entries. JDK 8 commonly keeps the default under jre/lib/security/cacerts; JDK 11+ commonly uses lib/security/cacerts. An explicit javax.net.ssl.trustStore, jssecacerts or a custom TrustManager can take precedence. Application servers may have their own trust configuration; the inbound HTTPS keystore is not necessarily the outbound trust store.

**Option 1 — vendor blocklist data (Zulu).** Azul supplies replacement files for existing Zulu installations. Follow its platform-specific download, checksum, destination and restart instructions [R3]. Its procedure covers blocked.certs/blacklisted.certs where supported. Do not copy a Zulu blocklist into arbitrary Oracle, IBM or other JDK distributions without applicable vendor confirmation.

**Option 2 — remove relevant roots from the effective store (Oracle-documented remedy).** Oracle documents this for JDK 8 and later [R4]. To reduce the scope of change, a managed copy may be used where the application supports an explicit trustStore:

1. Copy the effective store to a versioned application/platform location; preserve the original format and required enterprise roots.
2. List entries and verify the certificate fingerprints against the approved removal set. Oracle's documented aliases include emsigneccrootcag3 [jdk] and comodoaaaca [jdk], but aliases differ across distributions and custom stores.
3. Delete only the verified anchor entries from the copy. Repeat for each applicable approved alias:

```bash
/path/to/runtime/bin/keytool -delete \
  -keystore /opt/company/trust/app-cacerts \
  -alias 'VERIFIED_ALIAS'
```

4. Configure the consuming JVM before application startup:

```text
-Djavax.net.ssl.trustStore=/opt/company/trust/app-cacerts
```

Retain the correct trustStoreType and protected password configuration if needed. Do not assume a newer PKCS12 file is readable by a much older JDK; modifying a copy in its existing format reduces compatibility risk. A provider-specific store such as PKCS11 needs its own procedure.

5. Restart the relevant JVMs and validate. An application that creates its own SSLContext/TrustManager must be configured through that application's supported mechanism instead.

**Option 3 — revocation checking.** Oracle also documents com.sun.net.ssl.checkRevocation and com.sun.security.enableCRLDP [R4]. This is conditional on provider and application behaviour, CRL access and failure policy. Prefer a deterministic supported blocklist/store remedy for disconnected applications; do not declare success just because flags were added.

**Validate without upgrading.** Oracle links [CheckCerts.java](https://cr.openjdk.org/~mr/CheckCerts.java) [R14], described as working on JDK 8+. Obtain it from the vendor source, review it and run it using the target runtime and relevant store settings. JDK 8 requires compilation (javac then java); do not assume Java source-file launch works on 8. The vendor check supplements, rather than replaces, checks of custom application clients and normal integrations. Use the actual application HTTP client to validate representative emSign and AAA fixtures when required by Route B.

**JDK 6/7, IBM Java and proprietary application servers:** no universal incident-specific file replacement has been verified here. Some support an external trust store, but confirm exact JSSE/provider semantics and server support. Seek a vendor hotfix or a validated scoped store change; do not promise that a JDK 8 utility or another vendor's blocklist works. Track inability to validate as an exception rather than forcing a blind major-version upgrade.

### Node.js: older bundled roots and custom agents

**Discover.**

```bash
/path/to/node -p "JSON.stringify(process.versions)"
/path/to/node --help
```

Inspect startup flags and the selected CA-related environment variables only; avoid dumping the full environment. Check custom HTTPS agents, explicit ca options, TLS callbacks and HTTP libraries.

**Option 1 — switch to a managed OpenSSL store.** The Node 18 reference documents --use-openssl-ca as available since 6.11.0 and the CA-file/directory environment variables since 7.7.0 [S1]. This is an existing configuration feature, not a claim that all those old versions are vendor-supported or fixed.

For a Linux deployment where the actual build supports it, start the application with a vetted bundle and an explicitly controlled CA directory:

```bash
SSL_CERT_FILE=/opt/company/trust/approved-ca.pem \
SSL_CERT_DIR=/opt/company/trust/empty-ca-dir \
/path/to/node --use-openssl-ca app.js
```

Provision empty-ca-dir as a real empty directory owned by the platform team. This avoids unintentionally retaining a stale default CA directory alongside the new file. Scope these settings to the service; child processes may inherit them. Preserve other required launch arguments. Check for conflicting startup options and NODE_EXTRA_CA_CERTS that add trust back.

**Option 2 — configure the application's actual HTTPS agent.** Where the application exposes this configuration, supply an explicit approved ca bundle while retaining certificate and hostname verification. Example for Node 12+ using the core HTTPS client:

```javascript
const fs = require('fs');
const https = require('https');
const agent = new https.Agent({
  ca: fs.readFileSync('/opt/company/trust/approved-ca.pem'),
  rejectUnauthorized: true
});
// Supply this agent to the actual client/SDK through its supported configuration.
```

An explicit ca option replaces the normal CA selection for that TLS context. This only covers clients using that agent. Built-in fetch/Undici, a separate SDK, WebSocket library or Electron browser networking can use another configuration. Do not assume https.globalAgent changes every client.

**Do not use** NODE_EXTRA_CA_CERTS as a removal mechanism, NODE_TLS_REJECT_UNAUTHORIZED=0, or rejectUnauthorized:false. Do not prescribe recent --use-system-ca or newer CA APIs to an old runtime without checking availability.

**Validate.** Start a fresh process using the proposed service launch configuration. Through the application's actual client/agent, request approved emSign and AAA test endpoints and a legitimate control. A core-https probe is useful only for that client configuration; record the certificate error rather than treating any exception as a pass. Confirm the service restart really retained the environment/agent change.

For Electron or closed-source Node-packaged software, request the application's supported remedy. Do not assume a desktop environment variable changes the Chromium network verifier. [S1][S2]

### Python: keep the interpreter, change the correct bundle

**Discover using the exact virtual environment.**

```bash
/path/to/venv/bin/python -c "import sys,ssl; print(sys.executable); print(sys.version); print(ssl.OPENSSL_VERSION); print(ssl.get_default_verify_paths())"
/path/to/venv/bin/python -m pip show certifi requests
```

The ssl inspection applies to versions exposing those functions (get_default_verify_paths was introduced in Python 3.4). If pip/certifi is absent, use the relevant package manager/configuration instead; do not install packages merely to inspect the environment.

**Option 1 — change only certifi.** Where dependency constraints allow it, pin a compatible vendor-confirmed fixed certifi release in the lockfile and redeploy that environment. This need not upgrade Python, Requests or the application. Verify the chosen release still supports that interpreter. Do not hand-edit certifi's installed cacert.pem as the normal fix: its project does not support bundle modification and reinstall can overwrite it. [R5]

**Option 2 — Requests alternate bundle.** For supported Requests 2.x deployments, use the application's supported configuration or service-scoped environment:

```bash
REQUESTS_CA_BUNDLE=/opt/company/trust/approved-ca.pem \
/path/to/venv/bin/python app.py
```

An explicit request-level verify path is another documented option:

```python
response = requests.get(
    test_url,
    verify="/opt/company/trust/approved-ca.pem",
    timeout=10,
    allow_redirects=False
)
```

Use only approved diagnostic URLs without credentials. This snippet illustrates bundle selection; changing a diagnostic call does not fix an existing application. Configure its real Session/client. A custom adapter/context, trust_env=False or a prepared-request flow that does not merge environment settings can bypass the intended environment configuration. Requests documents the prepared-request caveat [R6].

**Option 3 — stdlib SSL clients.** Python 3.8 documentation provides a new context with an explicit CA file:

```python
import ssl
context = ssl.create_default_context(
    cafile="/opt/company/trust/approved-ca.pem"
)
# Pass this context to the actual SSL/HTTP client through its supported API.
```

Create a new context with the selected file; do not assume adding certificates to an already populated context removes previous roots. Keep certificate and hostname checks enabled. REQUESTS_CA_BUNDLE is not a universal Python switch: urllib, aiohttp, HTTPX and custom SDKs must be checked against their own version-specific trust configuration.

**Validate.** Exercise the same deployed interpreter and client wrapper, with the environment or explicit context actually used by the service. In Requests, a certificate-related SSLError is a candidate rejection result; inspect its cause. HTTP responses from the negative host mean TLS was accepted, while DNS/timeouts and unrelated protocol errors are inconclusive. Run a normal positive integration check and restart long-lived workers.

**Python 2 / very old Python 3:** do not assume current certifi can be installed or modern verification defaults apply. Obtain a supported distributor backport or confirm verification and hostname checking in the deployed client. A new bundle cannot repair a client that never verifies peers. This is a separate identified gap, not a reason to claim every application must upgrade immediately. [S3]

### Windows and .NET

For .NET Framework using normal Windows TLS validation, apply the Windows trust/distrust update through the platform team's approved distribution mechanism. This can leave the application and .NET Framework version unchanged. Use the shared Windows assurance record and confirm there are no permissive certificate callbacks or private trust mechanisms. [R2][S4]

Do not equate all .NET deployments with Windows: Linux/container .NET and libraries using their own chain policies need platform-specific assessment. Adding certificates to a custom store is not automatically a distrust policy.

Validate with the same HttpClient/handler or application operation under the relevant service identity. PowerShell Invoke-WebRequest uses PowerShell's own runtime and is not by itself proof of an unrelated application's behaviour. Restart/recycle affected services where necessary and test normal connections.

### Linux, curl and OpenSSL consumers

Use the distribution's supported CA-data/security update without a general application upgrade; Ubuntu explicitly documents targeted packages [R1]. Check distribution-specific Java synchronisation and preserve private CAs.

For an explicitly file-based curl consumer, an approved bundle can be selected for a diagnostic or supported application invocation:

```bash
curl -V
curl --cacert /opt/company/trust/approved-ca.pem \
  --connect-timeout 10 --max-time 20 \
  https://dv-g3b-aaa-emudhra.we-have-a-problem.com/
```

curl -V identifies the TLS backend. Schannel/native trust, an additional CA directory or application/libcurl overrides can change the result; inspect the actual backend and version before claiming the file is exclusive. Do not use --insecure. Test the emSign route as well. A successful curl test does not validate a JVM or another verifier on the host. [S5]

### Other stacks: scope the workaround before promising it

| Stack | Store-only option to investigate | Validation / limit |
| --- | --- | --- |
| Go services | Preferred: rebuild with Go 1.27.3/1.26.10 or later [R8]. Store-only: update the system bundle if the binary uses system roots; on applicable older Unix builds check SSL_CERT_FILE and SSL_CERT_DIR handling. | Restart and test the actual binary. Custom RootCAs/embedded pools can ignore system changes. Behaviour differs by OS and Go version; current Go 1.27 documentation explicitly notes changed override semantics, so do not project them onto old Windows/macOS binaries. A runtime denylist fix can require rebuilding the binary. [S6] |
| Rust/native-TLS versus rustls | Identify whether roots come from the native platform, a file, or a compiled dependency. | For embedded roots without a supported override, rebuilding the same application release with corrected dependency data may be necessary. Do not promise a generic environment-variable fix. |
| PHP OpenSSL clients | Configure openssl.cafile and check openssl.capath fallback in the effective service configuration. | Validate the actual PHP-FPM/Apache workload, not only CLI php.ini. PHP cURL is a separate client path and needs its own supported CA configuration. [S7] |
| nginx reverse proxy | Use proxy_ssl_trusted_certificate for the upstream bundle and verify proxy_ssl_verify is enabled, with correct upstream name/SNI policy. | Test the actual backend TLS leg in a representative configuration and reload safely. Replacing the site's frontend certificate does not fix upstream trust. Verification is off by default in the cited directive documentation. [S8] |

### Reusable validation and closure for all recipes

PKI/platform teams should validate their chosen remediation pattern centrally against the current approved emSign and AAA matrix. Include a valid connection as a positive control; distinguish affected-chain rejection from expiry, hostname, protocol, DNS and network failures. Offline tests must use the relevant validation engine, or have their coverage limits recorded.

For a standard configuration, application teams only confirm applicability, deployment coverage and absence of relevant overrides. For custom Route B cases, keep the selected configuration and representative result in the existing change/test record. Do not add per-instance hashing or a new test harness requirement for every application.

A store change is active only after the actual client uses it; a file on disk alone is insufficient. Check fresh connections after restart/reload, relevant mounted configuration and one representative deployment per materially different configuration. Do not make new outbound production access merely to reach a checker.

These recipes describe vendor procedures where explicitly labelled and otherwise documented configuration mechanisms adapted to this incident. Validate against the exact deployed build; no snippets in this appendix have been executed against the enterprise estate.

These options are a decision framework, not instructions already applied to the enterprise. Recheck the exact vendor advisory before implementation.

## References

**Related documents**

- [Playbooks](playbooks/README.md): the operational steps for each team
- [Action plan](ACTION_PLAN.md), [Prioritisation](RISK_TIERS.md), [Application validation](APPLICATION_VALIDATION.md), [CBOM and cross-signing](CBOM_AND_CROSS_SIGNING.md)

**Remediation references**

- [R1: Canonical targeted fixes and Java-store caveat](https://ubuntu.com/blog/emudhra-ca-compromise-fixes-available).
- [R2: Microsoft CTL and disallowed-certificate distribution](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates).
- [R3: Azul downloadable blocklist workaround](https://docs.azul.com/core/emudhra-workaround).
- [R4: Oracle JDK eMudhra guidance, updated 23 October](https://ops.java/emudhra).
- [R5: Certifi bundle and modification policy](https://github.com/certifi/python-certifi).
- [R6: Requests alternate CA-bundle configuration](https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification).
- [R7: Node.js CA-selection configuration](https://nodejs.org/api/cli.html).
- [R8: Go release history (1.27.3 and 1.26.10)](https://go.dev/doc/devel/release).
- [R9: webpki-roots versions on crates.io](https://crates.io/crates/webpki-roots/versions).
- [R10: certifi release history on PyPI](https://pypi.org/project/certifi/#history).
- [R11: Ubuntu ca-certificates bug 2171507](https://bugs.launchpad.net/ubuntu/+source/ca-certificates/+bug/2171507).
- [R12: Rescana fix-status summary, 20–21 October](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026).
- [R13: lilting.ch macOS curl, Node.js and uv tests](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys).
- [R14: Oracle-linked CheckCerts.java validation utility](https://cr.openjdk.org/~mr/CheckCerts.java).
- [R15: Microsoft Security Response Center: response to compromised eMudhra CA keys](https://www.microsoft.com/en-us/msrc/blog/2026/10/microsoft-response-to-compromised-emudhra-certificate-authority-keys).
- [R16: Node.js release index (security releases 20–21 October)](https://nodejs.org/dist/index.json).

**Stack documentation**

- [S1: Node 18 CLI — older CA-selection flags and environment variables](https://nodejs.org/docs/latest-v18.x/api/cli.html).
- [S2: Node TLS client CA options](https://nodejs.org/api/tls.html).
- [S3: Python 3.8 SSL context and default-store documentation](https://docs.python.org/3.8/library/ssl.html).
- [S4: Microsoft .NET Framework TLS guidance](https://learn.microsoft.com/en-us/dotnet/framework/network-programming/tls).
- [S5: curl certificate-store and backend guidance](https://curl.se/docs/sslcerts.html).
- [S6: Go x509 system certificate pool](https://pkg.go.dev/crypto/x509#SystemCertPool).
- [S7: PHP OpenSSL CA configuration](https://www.php.net/manual/en/openssl.configuration.php).
- [S8: nginx upstream certificate verification](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_trusted_certificate).
