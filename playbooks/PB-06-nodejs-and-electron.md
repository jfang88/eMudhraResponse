# PB-06 Node.js, Bun and Electron apps

**Owner:** application owner (services) · desktop team and vendor (Electron apps) · **Typical priority:** P1–P2 (susceptibility C: roots compiled into the runtime)

## Applies if

- A Node.js or Bun service, function, CLI or build tool makes outbound HTTPS
  calls.
- Or you run **Electron** desktop apps (Slack, VS Code, many vendor tools) or
  build your own.

Node compiles a copy of the Mozilla root list **into the binary** and doesn't
check revocation. Node.js published security releases on 20–21 October
(22.23.4, 24.22.1, 26.12.1), which third-party advisories list as the fix;
confirm in the Node.js security release notes. Bun's status is unconfirmed.
Until you're on a fixed release, use a configuration change.

## Check what the runtime trusts

```bash
/path/to/node -p "process.version"
# Does the built-in root list still contain the affected root? (Node 15.6+)
# Prints the subject if present. Node 22.22.0 still printed it on 26 Oct.
/path/to/node -e "const {X509Certificate}=require('crypto');for(const p of require('tls').rootCertificates){const s=new X509Certificate(p).subject;if(/emSign ECC Root CA - G3/.test(s))console.log(s.replace(/\n/g,', '))}"
# Which CA options does this build support?
/path/to/node --help | grep -E 'use-openssl-ca|use-system-ca|use-bundled-ca'
```

Also check the service's start-up flags and these variables (names and values
only, not the whole environment): `NODE_OPTIONS`, `NODE_EXTRA_CA_CERTS`,
`SSL_CERT_FILE`, `SSL_CERT_DIR`, `NODE_TLS_REJECT_UNAUTHORIZED`.

## Fix: pick the first option that applies

**Option 1: a fixed release.** Upgrade within your current major line to
22.23.4, 24.22.1 or 26.12.1 (or later) and redeploy. For Bun, use its fixed
release once confirmed.

**Option 2: use the OS store, which is already patched.**

- If `--use-system-ca` is listed by `--help` (recent releases), it makes Node
  use the platform verifier store (Windows/macOS) or the system bundle (Linux),
  so the OS fix (PB-02 or PB-03) applies.
- Otherwise use `--use-openssl-ca` (Linux) with a controlled bundle:

  ```bash
  SSL_CERT_FILE=/opt/company/trust/approved-ca.pem \
  SSL_CERT_DIR=/opt/company/trust/empty-ca-dir \
  /path/to/node --use-openssl-ca app.js
  ```

  `empty-ca-dir` must be a real, empty, platform-owned directory, so no stale
  default directory gets picked up.

**Option 3: set an explicit `ca` on the client the app really uses.**

```javascript
const agent = new (require('https').Agent)({
  ca: require('fs').readFileSync('/opt/company/trust/approved-ca.pem'),
  rejectUnauthorized: true
});
// Pass this agent to the real SDK/client. Built-in fetch (Undici), axios,
// got and cloud SDKs each have their own way to accept it.
```

## Electron apps

- Chromium-side traffic (the app's web content) uses the **OS verifier**, so
  it's protected once Windows or macOS is updated. Electron doesn't receive
  Chrome's CRLSet updates.
- **Node-side traffic** (auto-update checks, API calls from the main process)
  uses the **bundled Node roots** until the vendor ships an update.
- For third-party Electron apps: ask the vendor (PB-11) and record its
  statement. For apps we build: rebuild on a fixed Electron/Node version, or
  apply option 2 or 3 in the main process.

## Code checks

Search for `rejectUnauthorized: false`, `NODE_TLS_REJECT_UNAUTHORIZED=0`,
`checkServerIdentity` overrides and `strictSSL: false` (npm, older request
libraries).

## Verify

```bash
for h in dv-g3b-emudhra dv-g3b-aaa-emudhra dv-g3b-noct-emudhra dv-g3b-aaa-noct-emudhra; do
  /path/to/node [your-ca-flags] -e "require('https').get('https://$h.we-have-a-problem.com/',r=>{console.log('FAIL','$h',r.statusCode);process.exit()}).on('error',e=>console.log('CHECK','$h',e.code))"
done
```

`CHECK` must show a certificate error code (for example
`UNABLE_TO_GET_ISSUER_CERT_LOCALLY` or `CERT_REVOKED`), not `ENOTFOUND` or
`ETIMEDOUT`. This only tests core `https` with those flags. Repeat through the
app's real client, and restart the service so new flags take effect.

## Record

- Shared Node base images: a **Route A record** (version, flags, test result,
  deployments covered).
- Apps with a custom agent or explicit `ca`: **Route B**.
- Third-party Electron apps: long-tail record with the vendor's status.

## Watch out for

- **`NODE_EXTRA_CA_CERTS` only adds trust.** It never removes a root.
- An explicit `ca` **replaces** the defaults for that client. Include every
  root the app legitimately needs.
- Child processes inherit `NODE_OPTIONS` and the CA variables. Scope them to
  the service.
- npm, yarn and pnpm in build pipelines have the same exposure (PB-08).

## Escalate when

- No fixed release exists for your major line, and the app's client ignores both OS-store flags
  and explicit agents, on a P1 path.
- An Electron vendor won't commit to a date.

## References

**Related documents**

- [Team guide](README.md)
- [Remediation options: Node.js](../REMEDIATION_OPTIONS.md#nodejs-older-bundled-roots-and-custom-agents)

**External sources**

- [Node.js release index (security releases 20–21 October)](https://nodejs.org/dist/index.json)
- [Node.js: command-line options (CA selection)](https://nodejs.org/api/cli.html)
- [Node.js: TLS client CA options](https://nodejs.org/api/tls.html)
- [Rescana: fix-status summary, 20–21 October](https://rescana.com/post/emudhra-intermediate-ca-key-compromise-october-2026)
- [lilting.ch: macOS curl, Node.js and uv tests against the recovered keys](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
