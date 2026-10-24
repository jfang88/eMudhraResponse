# Application validation: find affected trust and prove remediation

**24 October 2026 | Recommended operational runbook**

## 1. Define the verification path

For each application, record the deployed executable/runtime, HTTP/TLS library, service identity, environment, image digest, network route and effective trust configuration. Cover outbound APIs, webhooks, package downloads, scheduled jobs and relevant backend TLS connections. A server's own certificate inventory is a separate check: it does not reveal what that server trusts as a client.

Test distinct verification paths separately. The same host can contain an OS store, Java store, Python virtual environments, bundled browser engines and a proxy with different trust decisions. Use the running workload's configuration; a developer shell or host-level curl may select another verifier.

## 2. Discover hidden and overridden stores

The following is an investigation checklist, not an automatic verdict or instruction to delete certificates.

| Stack / location | What the application owner should inspect |
| --- | --- |
| Linux / OpenSSL-based software | Effective CA file and directory; common locations include /etc/ssl/certs, /etc/ssl/cert.pem and /etc/pki. Inspect SSL_CERT_FILE / SSL_CERT_DIR only where the client honours them, service configuration and private bundles under application directories. |
| Windows | Machine and relevant service/user certificate stores, effective distrust policy and application-private bundles. Testing as an administrator does not prove behaviour under the service account. |
| Java | Actual deployed JRE/JDK, javax.net.ssl.trustStore, jssecacerts, cacerts and custom TrustManager/SSLContext implementations. The explicit trustStore or jssecacerts can override the expected default cacerts. [J] |
| Python | Actual virtual environment/interpreter, OpenSSL defaults, certifi location if installed, library CA overrides and custom SSLContext objects. Defaults alone do not prove what a higher-level client uses. [P] |
| Node / Electron | Runtime build, CA-selection flags, NODE_EXTRA_CA_CERTS and explicit TLS/HTTPS ca options. Explicit ca configuration can replace default roots; inspect application code and wrapper libraries. [N] |
| Go / Rust / native or packaged tools | Compiler/runtime and TLS-library versions, embedded certificate data, private root pools and custom validation logic. Updating the host bundle may not update a compiled dependency. |
| Containers / Kubernetes | Final image and all application environments; mounted Secrets/ConfigMaps/volumes; init scripts that copy certificates; sidecar/proxy trust; stale stores in persistent volumes. Avoid exposing secret values. |
| CI / deployment artefacts | Build scripts, vendored bundles, base images, bootstrap configuration and rollback artefacts that could restore an old store. |

Within an approved application directory, a read-only filename search can locate candidates:

```bash
rg --files --hidden --no-ignore /path/to/application |
  rg -i '(^|/)(cacerts|jssecacerts|cert\.pem|ca-bundle[^/]*|ca-certificates[^/]*)$|\.(pem|crt|cer|jks|p12|pfx)$'
```

This is not exhaustive: binary archives, embedded roots and inaccessible paths need separate examination. Certificate-looking files may be leaf certificates or private-key containers, not trust stores. Record locations and ownership; do not upload private keys or credentials.

For Python, use the deployed interpreter to inspect defaults:

```bash
/path/to/venv/bin/python -c 'import ssl; print(ssl.get_default_verify_paths())'
/path/to/venv/bin/python -c 'import certifi; print(certifi.where())'
```

The second command applies only if certifi is installed. Neither command proves that the application's chosen HTTP client uses that path.

## 3. Compare certificate identities and effective trust

Obtain the current [affected intermediates and cross-signs](https://emudhra.we-have-a-problem.com/impacted-intermediates.json) and [root reference](https://emudhra.we-have-a-problem.com/roots.json). Preserve the reference retrieval date and a hash of the copy used for assessment.

Parse candidate stores using their supported format tools. Compare certificate SHA-256 fingerprints and, where supported, public-key/SPKI identities against the affected set. Subject-name searches help discovery but are insufficient for a decision. Different certificates can share a subject; cross-signed certificates can share a key.

**No matching intermediate in a local store does not prove safety:** servers can supply intermediates that chain to a locally trusted root. Conversely, a certificate's presence does not prove it remains usable if the actual verifier enforces an effective blocklist or other distrust policy. Assess chain building and policy together.

Do not blanket-delete every eMudhra or Sectigo certificate. Use vendor-specific remedies and test their availability impact. Removing a broad root may affect unrelated legitimate services, and removing only one path may leave an alternate chain trusted.

## 4. Audit verification bypasses

Inspect code and configuration for disabled certificate/hostname verification, permissive callbacks and fallback retries that suppress TLS errors. Examples worth review include verify=False, rejectUnauthorized:false, NODE_TLS_REJECT_UNAUTHORIZED=0, InsecureSkipVerify, trust-all Java managers and curl -k/--insecure. Confirm whether each is active in deployed code; a text match alone is not a finding.

Test through the same client wrapper used for real business calls. A bare runtime request can pass while an application-specific client still bypasses verification.

## 5. Perform negative and positive tests

Use the incident site's [current test matrix](https://emudhra.we-have-a-problem.com/#check), covering affected TLS intermediates and alternative chains, including the AAA cross-sign route. Include No-CT cases; a CT-only rejection does not demonstrate removal of CA trust.

The following are initial curl probes only, not a full application test suite:

```bash
curl --connect-timeout 10 --max-time 20 https://dv-g3b-emudhra.we-have-a-problem.com/
curl --connect-timeout 10 --max-time 20 https://dv-g3b-aaa-emudhra.we-have-a-problem.com/
curl --connect-timeout 10 --max-time 20 https://dv-g3b-noct-emudhra.we-have-a-problem.com/
curl --connect-timeout 10 --max-time 20 https://dv-g3b-aaa-noct-emudhra.we-have-a-problem.com/
```

Do not add -k, credentials, cookies or business data. Use an approved diagnostic client or harness that exercises the application's actual library and configuration. For production endpoints, follow ordinary change controls; do not repoint live business transactions at test hosts.

| Observation | Classification / next step |
| --- | --- |
| TLS accepted and an HTTP response received from a test host | Fail for that verification path, even if the HTTP status is an error. |
| Explicit rejection attributable to the affected chain and intended trust policy | Pass for that tested chain and verifier; retain error evidence. |
| DNS failure, timeout, firewall deny or proxy-generated block page | Inconclusive for application trust. |
| Only an expired certificate, hostname mismatch or unrelated validation error causes failure | Inconclusive for incident remediation; verify the current fixture and rejection reason. |
| CT rejection alone | That test is protected by CT, but does not establish that the compromised chain is distrusted. |
| Normal approved HTTPS integration also fails | Investigate regression before closure. |

A browser check is useful for browser scope only. If TLS inspection presents a corporate-issued certificate, document that the proxy made the upstream trust decision; do not credit the application's own store. Test other authorised routes or use offline validation rather than bypassing enterprise controls.

For restricted environments, use the reference site's [offline chain bundle](https://emudhra.we-have-a-problem.com/chains.zip) and its included verification instructions. An OpenSSL result proves only the tested OpenSSL policy and store. Java, Windows or custom verifiers need equivalent tests through their actual validation engines. Never install test certificates as trusted roots.

## 6. Activate and verify the deployed fix

After patching, rebuild/redeploy as needed and establish a fresh connection to avoid reusing an existing pooled TLS session. Confirm restarted processes use the new artefact and configuration. Verify all relevant replicas, sites, jobs and service identities, or document justified equivalence groups.

Check certificates presented by business endpoints separately for chains affected by distrust. Perform normal integration tests after any replacement. Review mTLS client authentication separately where public CA trust is configured; do not assume a server-authentication test covers client authentication.

## 7. Evidence and acceptance

Record application/asset ID, owner, timestamp, deployed digest/version, verifier, trust-store location and SHA-256 hash where file-backed, effective policy, proxy route, test chains, exact errors, positive control, scope and reviewer.

Close only after inventory, deployment and behavioural evidence agree. A certificate expiry scanner, SBOM, package version or absence of a known name is supporting evidence, not standalone proof. Keep unexplained results open.

## References

- [Incident reference, cross-sign inventory, maintained fixes and test fixtures](https://emudhra.we-have-a-problem.com/).
- [Cloudflare: validate the actual application TLS path](https://blog.cloudflare.com/emudhra-certificate-compromise/).
- [J: Oracle JSSE trust-store selection](https://docs.oracle.com/en/java/javase/26/security/java-secure-socket-extension-jsse-reference-guide.html).
- [P: Python ssl defaults and contexts](https://docs.python.org/3.12/library/ssl.html).
- [N: Node.js CA configuration and environment variables](https://nodejs.org/api/cli.html).

Discovery steps and acceptance criteria are proposed enterprise procedures. Commands are illustrative read-only checks; they have not been executed against the enterprise estate. Consult documentation for the exact deployed runtime version.
