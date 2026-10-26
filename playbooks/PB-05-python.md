# PB-05 Python applications, jobs and tools

**Owner:** application owner · **Typical priority:** P1–P2 (susceptibility C when certifi is used)

## Applies if

Python code makes outbound HTTPS calls: `requests`, `httpx`, `urllib3`,
`aiohttp`, `boto3`/cloud SDKs, Airflow, Ansible, data pipelines, Lambda or
Azure Functions packages, Conda environments.

Python's default stacks don't check revocation, so the vendors' browser and OS
blocks don't help. What matters is **which CA bundle** the client loads.

## Find the bundle the code actually uses

Run these with the **deployed** interpreter or virtual environment:

```bash
/path/to/venv/bin/python - <<'EOF'
import sys, ssl
print("python :", sys.executable, sys.version.split()[0])
print("openssl:", ssl.OPENSSL_VERSION)
print("default:", ssl.get_default_verify_paths())
try:
    import certifi; print("certifi:", certifi.__version__, certifi.where())
except ImportError:
    print("certifi: not installed")
EOF
```

- `requests` (and most SDKs built on it) uses **certifi** unless
  `REQUESTS_CA_BUNDLE`, `verify=` or a custom adapter says otherwise.
- `ssl.create_default_context()` and `urllib` use the **OpenSSL default
  paths**, which are usually the OS bundle, so fix the host via PB-03.
- Conda environments carry their own `ca-certificates` and `certifi` packages
  (`conda list ca-certificates certifi`).

## Fix: pick the first option that applies

**Option 1: update certifi only.** Pin **certifi 2026.10.18 or later** in your
lockfile, then rebuild and redeploy. On 26 Oct we checked: 2026.7.22 still
contains emSign ECC Root CA - G3, and 2026.10.18, .19 and .23 do not. Note that
2026.10.23 also drops emSign Root CA - C1 and emSign ECC Root CA - C3, which
matters for availability (PB-13). This doesn't
require upgrading Python, `requests` or the app. Don't hand-edit certifi's
`cacert.pem`: it's unsupported and is overwritten on reinstall.

**Option 2: point the client at a managed bundle** (when you can't change
dependencies yet):

```bash
# service-scoped environment, for requests-based code
REQUESTS_CA_BUNDLE=/opt/company/trust/approved-ca.pem /path/to/venv/bin/python app.py
```

Or in code: set `session.verify = "/opt/company/trust/approved-ca.pem"` on the
app's own `requests.Session`, or `ssl.create_default_context(cafile=...)` passed to the client. Create a
**new** context; adding certificates to an existing one doesn't remove trust.

**Option 3: use the OS trust store** (Python 3.10+): the `truststore` package
makes `ssl` use the platform verifier (Windows/macOS native, Linux OpenSSL
store), so the OS fix applies. It's a code change: test it in non-production.

## Code checks

Search for `verify=False`, `ssl._create_unverified_context`,
`CERT_NONE`, `check_hostname = False`, `trust_env=False` (which ignores
`REQUESTS_CA_BUNDLE`) and custom `HTTPAdapter` or `SSLContext` set-ups.

## Verify

```bash
/path/to/venv/bin/python - <<'EOF'
import requests
for h in ["dv-g3b-emudhra","dv-g3b-aaa-emudhra","dv-g3b-noct-emudhra","dv-g3b-aaa-noct-emudhra"]:
    try:
        r = requests.get(f"https://{h}.we-have-a-problem.com/", timeout=15, allow_redirects=False)
        print("FAIL ", h, r.status_code)
    except requests.exceptions.SSLError as e:
        print("PASS?", h, str(e)[:120])   # confirm the reason is certificate trust
    except requests.exceptions.RequestException as e:
        print("INCONCLUSIVE", h, type(e).__name__)
EOF
```

This tests plain `requests` with the environment's settings. Repeat through
the app's real client or session if it configures TLS itself. Restart
long-lived workers (Gunicorn, Celery, Airflow) afterwards.

## Record

- Shared base images or golden venvs: a **Route A record** (certifi version,
  test results, which images or deployments carry it).
- Apps with explicit bundles or custom contexts: **Route B**.

## Watch out for

- **The new certifi removes a root**, so legitimate partners under emSign ECC
  Root CA - G3 will break. Check PB-13 first.
- **Serverless**: the provider updates the runtime, but certifi vendored into
  *your* deployment package is yours to update.
- **Python 2 or very old Python 3** may not install a current certifi, or may
  not verify at all. Treat as an exception, not a quick fix.
- `pip` itself uses its own vendored certifi (or truststore in recent
  versions). That matters for build pipelines (PB-08).

## Escalate when

- A P1 integration uses `verify=False` in production.
- Dependency pins block the certifi update and no explicit bundle option
  exists.

## References

**Related documents**

- [Team guide](README.md)
- [Remediation options: Python](../REMEDIATION_OPTIONS.md#python-keep-the-interpreter-change-the-correct-bundle)

**External sources**

- [PyPI: certifi release history](https://pypi.org/project/certifi/#history)
- [certifi: bundle and modification policy](https://github.com/certifi/python-certifi)
- [Requests: SSL certificate verification](https://requests.readthedocs.io/en/latest/user/advanced/#ssl-cert-verification)
- [truststore: use the OS trust store from Python](https://pypi.org/project/truststore/)
