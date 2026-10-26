# PB-09 Proxies, gateways, load balancers and TLS inspection

**Owner:** network / security engineering · **Typical priority:** **P1 for TLS-inspection and egress proxies**; reverse proxies by backend impact

## Applies if

You operate any device or service that **opens TLS connections on behalf of
others**:

- **Forward or TLS-inspection proxies, SWG/SASE** (on-premises or cloud):
  decrypt and re-sign outbound traffic.
- **Reverse proxies, load balancers, WAF, API gateways, ingress controllers,
  service-mesh egress gateways**: originate TLS to backends or external APIs.

## Why the inspection proxy comes first

For inspected traffic, the proxy **is** the verifier. The client, browser
included, only sees the proxy's re-signed certificate, so Chrome's CRLSet,
Firefox's OneCRL and Apple's and Microsoft's blocks **never see the real
chain**. The flip side: fix the proxy and every inspected client is protected
on that path, including non-browser software.

## Fix: TLS-inspection and forward proxies

1. **Update the proxy's upstream trust store** to the vendor's release that
   blocks the intermediates or removes emSign ECC Root CA - G3 (and handles
   the AAA cross-sign route). Get the vendor's written statement (PB-11).
2. **Set the upstream-certificate policy to *block*** for untrusted, revoked
   or unverifiable upstream certificates. Don't use "allow", "warn and
   continue", or "re-sign with an untrusted CA" (decision D-07).
3. **Revocation checking**: if the product supports CRL/OCSP for upstream
   certificates, turn it on, decide fail-open versus fail-closed, and record
   the decision.
4. **Bypass lists**: traffic exempted from inspection (pinned apps, banking,
   health) goes back to the client's own verifier. Note this in the PB-01
   records.

## Fix: reverse proxies, load balancers, gateways

- Find out whether **upstream verification is on at all**. Several products
  don't verify backends by default. If it's off, the backend leg is already
  unauthenticated: record that as a pre-existing weakness, and turn on
  verification with a scoped CA bundle where the backend is external.
- Where it's on, point it at a **managed bundle** without the affected root,
  or at the platform store once that's updated.

| Product | Setting to check |
|---|---|
| nginx | `proxy_ssl_verify on;` and `proxy_ssl_trusted_certificate` (verification is off by default) |
| HAProxy | `server ... ssl verify required ca-file /path/bundle.pem` |
| Envoy / Istio egress | `validation_context.trusted_ca` (or the image's system CA when unset) |
| F5, Citrix ADC, cloud LBs and API gateways | Server-side SSL profile: CA bundle and "verify server certificate"; for SaaS gateways, ask the provider (PB-11) |

## Verify

- **Inspection proxy**: from a client behind it, request each
  [PB-00](PB-00-test-and-evidence.md) test host. **Pass** means a proxy
  block page or connection refusal *because of the upstream certificate*
  (check the proxy log for the reason). **Fail** means the page loads with a
  certificate re-signed by the corporate CA.
- **Reverse proxy/gateway**: configure a test route to one test host in a
  non-production instance. It must refuse.
- Then run a positive control and normal service checks.

## Record

- One **Route A record per proxy platform and policy** (version, trust-store
  release, upstream policy setting, test result, sites covered).
- Proxy logs showing upstream issuers are useful to the SOC (PB-14) and the
  availability track (PB-13). Share the query.

## Watch out for

- A **proxy block page counts as inconclusive** for an application test but
  as a **pass** for the proxy itself.
- Split tunnels and direct-to-internet egress bypass the proxy. Those clients
  rely on their own verifier (PB-01, runtime playbooks).
- Cloud SWG/SASE: the vendor updates the trust store, but **your tenant
  policy** decides block versus warn. Check both.

## Escalate when

- The inspection proxy accepts any test host (this is a P1 incident path for
  every inspected client).
- The vendor has no trust-store update and the product can't take a custom
  bundle.

## References

**Related documents**

- [Team guide](README.md)
- [Remediation options: nginx](../REMEDIATION_OPTIONS.md#other-stacks-scope-the-workaround-before-promising-it)
- [Decision D-07 (upstream policy)](../ACTION_PLAN.md#appendix-a-decision-register)

**External sources**

- [nginx: upstream certificate verification](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_ssl_trusted_certificate)
- [HAProxy: server SSL verify and ca-file options](https://docs.haproxy.org/2.8/configuration.html#5.2-verify)
- [Cloudflare: 8 eMudhra certificates are compromised](https://blog.cloudflare.com/emudhra-certificate-compromise/)
