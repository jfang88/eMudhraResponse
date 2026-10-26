# PB-07 Go, Rust, PHP, Ruby and other runtimes

**Owner:** application owner · **Typical priority:** P1–P2 (Go and Rust binaries are often susceptibility C: fixed only by rebuild)

## Applies if

You build or run Go or Rust binaries, PHP (FPM, Apache mod_php, Composer),
Ruby (Rails, Bundler), or other runtimes that make outbound TLS calls.

## Go

**How Go chooses roots.** On Linux it reads the system bundle. On Windows and
macOS it uses the platform verifier. A binary can also carry embedded roots
(`golang.org/x/crypto/x509roots/fallback`, or a CA file copied into a
`scratch`/distroless image) or set its own `RootCAs`.

**Fix.**

1. **Rebuild with Go 1.27.3 or 1.26.10 or later.** These releases reject any
   chain containing one of the eight keys, whatever the roots. A binary is
   only protected once rebuilt. Go 1.25 and older get no fix.
2. If you can't rebuild yet: on Linux, update the system bundle (PB-03), and
   for scratch or distroless images, rebuild the image with the updated CA
   file (PB-08). This removes emSign ECC Root CA - G3 but doesn't block the
   keys directly.
3. If the binary uses `x509roots/fallback`, update that module and rebuild.

**Check.**

```bash
go version /path/to/binary            # Go toolchain the binary was built with
go version -m /path/to/binary | grep -E 'x509roots|crypto'   # embedded root modules
```

Third-party Go tools (Terraform providers, kubectl plugins, exporters,
agents) are vendor-controlled. Track them in PB-11.

## Rust

| Crate in `cargo tree` | Where roots come from | Fix |
|---|---|---|
| `webpki-roots` | Compiled-in Mozilla list | Update to **1.0.10 or later** (1.0.9 still contains emSign ECC Root CA - G3; checked 26 Oct), then rebuild |
| `rustls-native-certs` / `rustls-platform-verifier` | OS store | Fix the OS (PB-02 or PB-03); rebuild not required |
| `native-tls` / `openssl` | OpenSSL, SChannel or Security.framework | Fix the OS |

```bash
cargo tree -i webpki-roots     # who pulls in the compiled-in roots
```

## PHP

- The OpenSSL stream and cURL extensions use `openssl.cafile`/`openssl.capath`
  and `curl.cainfo` from **the php.ini the workload loads**. FPM and Apache
  often load a different file from the CLI (check with `php --ini` and the FPM
  pool config).
- If those are unset, the system bundle applies (PB-03).
- **Composer** uses `composer/ca-bundle`, which prefers the system bundle and
  falls back to its own. On 20–21 Oct a Composer fix was reported as pending.
  Update `composer/ca-bundle` when available, and make sure the system bundle
  is found.

## Ruby

- Ruby's `OpenSSL` uses the system default paths. Check with
  `ruby -ropenssl -e 'p OpenSSL::X509::DEFAULT_CERT_FILE, OpenSSL::X509::DEFAULT_CERT_DIR'`.
  Fix the OS (PB-03).
- Look for `SSL_CERT_FILE` overrides, gems that bundle their own CA file, and
  `verify_mode: OpenSSL::SSL::VERIFY_NONE`.

## Verify

Run the [PB-00](PB-00-test-and-evidence.md) test hosts **through the
application's real client**, from the deployed binary or container. Expect a
certificate error (Go: `x509: certificate signed by unknown authority` or a
rejection naming the blocked key), never a response. Then restart and run a
normal business call.

## Record

- Rebuilt binaries: **Route B** with the toolchain version and test result. A
  shared build pipeline that enforces Go ≥ 1.27.3/1.26.10 can be a **Route A**
  record for everything it builds.
- PHP/Ruby on standard hosts: reference the PB-03 host record; Route B only
  for custom CA settings.

## Watch out for

- **Static binaries outlive their build.** Find old ones via deployment
  manifests and image registries, not just source repositories.
- `InsecureSkipVerify: true` (Go), `danger_accept_invalid_certs` (Rust
  reqwest) and `CURLOPT_SSL_VERIFYPEER => false` (PHP) bypass everything.

## Escalate when

- A P1 Go service can't be rebuilt (no source, or a vendor binary).
- A Rust or Go binary embeds roots and the owner is unknown.

Depth: [REMEDIATION_OPTIONS: Other stacks](../REMEDIATION_OPTIONS.md#other-stacks-scope-the-workaround-before-promising-it).
