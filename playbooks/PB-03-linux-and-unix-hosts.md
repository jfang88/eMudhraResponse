# PB-03 Linux, Unix and macOS command-line tools

**Owner:** Linux/Unix platform team · **Typical priority:** P1–P3 depending on what runs on the host

## Applies if

The host has software that uses the **operating-system CA bundle** through
OpenSSL, GnuTLS, NSS or p11-kit: `curl`, `wget`, `git`, package managers,
Ruby, PHP, .NET on Linux, Go binaries that use system roots, Python not using
certifi, and Node with `--use-openssl-ca`.

This also covers **macOS command-line tools**. `/usr/bin/curl` uses
`/etc/ssl/cert.pem`, not the Keychain, and was shown to accept the affected
chain after Apple's block.

Java, certifi, Node's built-in roots and compiled Go/Rust roots are **separate
stores**. Fix the host bundle here, then follow their playbooks.

## What the fix looks like

Current Mozilla-derived bundles fix this by **removing emSign ECC Root CA -
G3** (NSS 3.131.1 does the same). AAA Certificate Services is no longer
trusted for TLS in current Mozilla-derived bundles, so the AAA route mainly
matters on older or vendor-modified bundles. Check yours.

## Fix

**Option 1: the distribution's CA update (preferred).**

| Family | Command (review before running) |
|---|---|
| Ubuntu | `sudo apt-get update && sudo apt-get install --only-upgrade ca-certificates libnss3` (Canonical documents targeted packages; see its Java-store note) |
| Debian | `sudo apt-get update && sudo apt-get install --only-upgrade ca-certificates`. Check the [Debian security tracker](https://security-tracker.debian.org/); the update was still pending on 20 Oct. |
| RHEL, Rocky, Alma, Fedora, Amazon Linux | `sudo dnf upgrade ca-certificates` (or `yum`), then `sudo update-ca-trust` |
| SUSE | `sudo zypper update ca-certificates-mozilla` |
| Alpine (often in containers) | `apk upgrade ca-certificates` |

**Option 2: remove the root locally while the package is pending.** This is a
documented local configuration change. Version it in configuration
management.

- **Debian/Ubuntu**: in `/etc/ca-certificates.conf`, change the line
  `mozilla/emSign_ECC_Root_CA_-_G3.crt` to
  `!mozilla/emSign_ECC_Root_CA_-_G3.crt`, then run
  `sudo update-ca-certificates --fresh`.
- **RHEL family**: copy the root certificate (from the Security/PKI reference)
  into `/etc/pki/ca-trust/source/blocklist/` (`blacklist/` on older
  releases), then run `sudo update-ca-trust extract`.
- Confirm by SHA-256 fingerprint, not subject name. Don't remove other emSign
  or Sectigo roots.

**macOS CLI**: use Homebrew `curl`/`openssl` with an updated
`ca-certificates` (`brew upgrade ca-certificates`), or wait for Apple's update
to `/etc/ssl/cert.pem`. For scripts on P1 paths, pass an approved bundle
explicitly (`curl --cacert /opt/company/trust/approved-ca.pem`).

## Verify

```bash
# Is the root still in the system bundle? (Debian/Ubuntu: symlink names)
ls /etc/ssl/certs | grep -i emsign
# RHEL family (p11-kit):
trust list | grep -i -A2 'emSign ECC Root CA - G3'
# Then the behavioural test with the host's curl (and note the TLS backend):
curl -V | head -1
```

Run the [PB-00](PB-00-test-and-evidence.md) curl loop from this host.
Restart long-running services that loaded the bundle at start-up, then test a
normal business connection.

## Record

- Platform team: a **Route A assurance record** per distribution and version
  (package version, test result, coverage from your patch tooling, local
  removals applied).
- App owners on standard hosts: reference it, and confirm the app uses the
  system bundle and not its own (tracker field 3).

## Watch out for

- **Java on Debian/Ubuntu** uses `/etc/ssl/certs/java/cacerts`, which is
  generated separately. Canonical documents a synchronisation issue (PB-04).
- **Root removal breaks legitimate sites** under emSign ECC Root CA - G3.
  Check the availability list (PB-13) before rolling out.
- `SSL_CERT_FILE`, `SSL_CERT_DIR`, `CURL_CA_BUNDLE` or app config can point
  somewhere else. Check the service's environment (variable names only; don't
  dump secrets).
- Golden images, AMIs and VM templates need the same change, or new builds
  will reintroduce the root.

## Escalate when

- The package isn't available and local removal is blocked by policy on a P1
  host.
- Removing the root breaks a business-critical partner (raise an exception;
  don't silently restore it).

## References

**Related documents**

- [Team guide](README.md)
- [Remediation options: Linux, curl and OpenSSL](../REMEDIATION_OPTIONS.md#linux-curl-and-openssl-consumers)

**External sources**

- [Canonical: eMudhra CA compromise fixes available](https://ubuntu.com/blog/emudhra-ca-compromise-fixes-available)
- [Launchpad: Ubuntu ca-certificates bug 2171507](https://bugs.launchpad.net/ubuntu/+source/ca-certificates/+bug/2171507)
- [Debian security tracker](https://security-tracker.debian.org/)
- [curl: certificate stores and TLS backends](https://curl.se/docs/sslcerts.html)
- [lilting.ch: macOS curl, Node.js and uv tests against the recovered keys](https://lilting.ch/en/articles/emudhra-weak-intermediate-ca-keys)
