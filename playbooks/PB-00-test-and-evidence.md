# PB-00 Test and evidence (read once, all teams)

**Owner:** Security/PKI · **Used by:** every playbook

## Applies if

You need to prove that a verifier rejects the affected chains, or you need to
record evidence for closure. It always applies to Route B and to the platform
teams who publish Route A assurance records.

## The test matrix

Security/PKI maintains the approved matrix. The incident reference site
publishes four TLS test hosts covering both chain families, with and without
Certificate Transparency (CT):

| Host | Route | Why it matters |
|---|---|---|
| `dv-g3b-emudhra.we-have-a-problem.com` | emSign | Basic case |
| `dv-g3b-aaa-emudhra.we-have-a-problem.com` | Sectigo AAA cross-sign | Matters on Windows, Java, older or vendor bundles |
| `dv-g3b-noct-emudhra.we-have-a-problem.com` | emSign, no CT | Shows the verifier rejects the chain itself, not only missing CT |
| `dv-g3b-aaa-noct-emudhra.we-have-a-problem.com` | AAA, no CT | As above, AAA route |

Always pair them with a **positive control**: a legitimate public site your
service normally reaches, ideally one whose certificate is issued by Sectigo
(this proves you haven't broken legitimate AAA-dependent chains).

Quick probe from the workload's own environment (not your laptop):

```bash
for h in dv-g3b-emudhra dv-g3b-aaa-emudhra dv-g3b-noct-emudhra dv-g3b-aaa-noct-emudhra; do
  echo "== $h"
  curl -sS -o /dev/null -w 'HTTP %{http_code}\n' --connect-timeout 10 --max-time 20 \
    "https://$h.we-have-a-problem.com/"
done
```

`HTTP 000` preceded by a certificate error (usually `curl: (60)`; some TLS
backends report `(35)`) is a pass. Any other HTTP code is a fail. This only tests **curl's** verifier. Your application may use another one.
Repeat the test through the application's own HTTP client (see your runtime
playbook).

## How to read the result

| What you see | Verdict for that verifier |
|---|---|
| An HTTP status code (even 404 or 500) from a test host | **Fail.** TLS was accepted. |
| A certificate error naming revocation, an untrusted issuer, or a distrusted or unknown CA | **Pass** for that chain. Keep the error text. |
| DNS failure, timeout, connection refused, firewall deny | **Inconclusive.** Fix the network path or test offline. |
| A proxy block page | **Inconclusive for the application**; **pass for the proxy** if the proxy blocked because of the upstream certificate (see PB-09). |
| Only expiry or hostname mismatch | **Inconclusive.** The fixture may be stale; ask Security/PKI. |
| Rejected only on the no-CT host | CT protects you, but the chain itself is still trusted. Treat as **Fail** for non-browser clients. |
| The positive control also fails | **Regression.** Fix before closing. |

## When you can't reach the test site

- Use the offline chain bundle from the reference site with **the verifier your
  application actually uses** (OpenSSL proves only OpenSSL).
- Or test in a representative environment with documented equivalence.
- Do not open production egress just for the test (action plan rule).
- Security/PKI may provide an internal test service (decision D-13).

## Record

Put the details in the existing change or test record and link it from
tracker field 5:

- Verifier (runtime and version, library, trust-store path), service identity, host or image digest
- Date and time, hosts tested, result per host, error text, positive control result
- Limitations: what you couldn't test and why

You don't need certificate hashes for a standard platform fix that is covered
by a Route A assurance record.

## Escalate when

- A test host returns HTTP content after you applied a fix.
- The positive control fails and you can't restore it without restoring the
  affected trust.
- You see a certificate chaining to an affected intermediate anywhere outside
  the test hosts. That is a possible forgery: call the SOC (PB-14).

## References

**Related documents**

- [Team guide](README.md)
- [Application validation §5: negative and positive tests](../APPLICATION_VALIDATION.md#5-perform-negative-and-positive-tests)
- [Completion tracker](../COMPLETION_TRACKER.md)
- [Decision D-13 (internal test service)](../ACTION_PLAN.md#appendix-a-decision-register)

**External sources**

- [Incident reference: impact, maintained package list and test fixtures](https://emudhra.we-have-a-problem.com/)
- [Incident reference: current test matrix](https://emudhra.we-have-a-problem.com/#check)
- [Incident reference: offline chain bundle](https://emudhra.we-have-a-problem.com/chains.zip)
- [Incident reference: affected intermediates and cross-signs (JSON)](https://emudhra.we-have-a-problem.com/impacted-intermediates.json)
- [curl: certificate stores and TLS backends](https://curl.se/docs/sslcerts.html)
