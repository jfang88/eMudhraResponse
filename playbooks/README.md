# eMudhra response playbooks: start here

**Version 0.1.0 · 26 October 2026 · Proposed procedures, not executed changes**

Find your playbook below, run its *Applies if* test, and follow it. Each playbook
has the same six headings:

1. **Applies if**: a yes/no test. If nothing applies, record Route C with the reason.
2. **Fix**: options in order of preference. Use the first one that works for you.
3. **Verify**: prove the fix is active in the running workload.
4. **Record**: what to put in the six-field tracker (workbook sheet *Service tracker*).
5. **Watch out for**: the mistakes we expect.
6. **Escalate when**: when to stop and call Security/PKI or the incident lead.

Every team should read [PB-00 Test and evidence](PB-00-test-and-evidence.md) once.

---

## Find it by team

| Your team | Start with | Then, if relevant |
|---|---|---|
| Desktop / end-user computing | [PB-01 Browsers and desktop OS](PB-01-browsers-and-desktop-os.md) | PB-06 (Electron apps), PB-11 (desktop agents), PB-12 (application control) |
| Windows server / platform | [PB-02 Windows and .NET](PB-02-windows-and-dotnet.md) | PB-04 (Java on Windows), PB-12 |
| Linux / Unix platform | [PB-03 Linux and Unix hosts](PB-03-linux-and-unix-hosts.md) | PB-04, PB-05 |
| Application owners and developers | Your runtime: [Java](PB-04-java.md) · [Python](PB-05-python.md) · [Node/Electron](PB-06-nodejs-and-electron.md) · [Go/Rust/PHP/Ruby](PB-07-go-rust-php-ruby.md) · [.NET](PB-02-windows-and-dotnet.md) | [PB-16 Identity and mTLS](PB-16-identity-and-mtls.md), PB-08 if containerised |
| Container / DevSecOps / CI-CD | [PB-08 Containers, Kubernetes and CI/CD](PB-08-containers-kubernetes-and-cicd.md) | Runtime playbook for each image |
| Network / proxy / WAF / load balancer | [PB-09 Proxies, gateways and TLS inspection](PB-09-proxies-gateways-and-tls-inspection.md) | PB-11 for vendor appliances |
| Mobile / MDM / mobile app teams | [PB-10 Mobile](PB-10-mobile.md) | PB-16 |
| Supplier, vendor and SaaS owners | [PB-11 Vendor products, appliances and suppliers](PB-11-vendor-products-appliances-and-suppliers.md) | — |
| Endpoint security / software distribution | [PB-12 Code signing and application control](PB-12-code-signing-and-application-control.md) | PB-14 |
| PKI / certificate management | [PB-13 Our certificates and availability](PB-13-our-certificates-and-availability.md) | PB-00 (approved test matrix), PB-09 |
| SOC / incident response | [PB-14 SOC threat hunt](PB-14-soc-threat-hunt.md) | PB-12 |
| Service desk / support | [PB-15 Service desk](PB-15-service-desk.md) | — |
| Identity / IAM / API platform | [PB-16 Identity and mTLS](PB-16-identity-and-mtls.md) | Runtime playbook |

## Find it by what you run

| If your system… | Playbook |
|---|---|
| Is a managed browser, or relies only on the Windows or macOS trust store | PB-01, PB-02 |
| Runs `java`, an app server (Tomcat, WebLogic, JBoss, WebSphere), Kafka, Elasticsearch, Jenkins, Artifactory, Nexus | PB-04 |
| Runs Python, uses `requests`/`httpx`/`urllib3`, `certifi`, Conda, Airflow, Ansible | PB-05 |
| Runs Node.js, Bun, or an Electron desktop app (Slack, VS Code, etc.) | PB-06 |
| Is a compiled Go or Rust binary, or PHP/Composer or Ruby/Bundler | PB-07 |
| Is .NET Framework, .NET on Linux, PowerShell, or IIS | PB-02 |
| Uses `curl`, `wget`, OpenSSL or the distro CA bundle on Linux, macOS CLI or BSD | PB-03 |
| Ships as a container image, Helm chart, Lambda/Function, or runs on a CI runner | PB-08 |
| Is an nginx, HAProxy, Envoy, F5, API gateway, WAF or forward/inspection proxy | PB-09 |
| Is a vendor appliance, SaaS, OT/IoT device, agent or packaged product | PB-11 |
| Validates JWTs, SAML assertions or OAuth tokens, or accepts client certificates | PB-16 |
| Presents a certificate issued by eMudhra/emSign, or calls a partner that does | PB-13 |

## Priority in one table

Priority = **impact** (H/M/L, using the pack's tier definitions) × **verifier
susceptibility**. Pending decision D-01, use it to order your work.

| | A: vendor-pushed block or CT enforced | B: patched OS store (confirm) | C: own or compiled bundle | D: embedded or vendor-only | Unknown |
|---|---|---|---|---|---|
| **H** | P3 | **P1** | **P1** | **P1** + exception | **P1 to assess** |
| **M** | P4 | P2 | P2 | P2 + vendor tracking | P2 to assess |
| **L** | P4 | P3 | P3 | P3 + vendor tracking | P3 to assess |

## Rules that apply everywhere

- **Deleting a cached intermediate is not distrust.** A server can send it again.
- **Adding certificates never removes trust.** For example, `NODE_EXTRA_CA_CERTS`
  only adds.
- **Never turn off verification** (`verify=False`, `-k`, `rejectUnauthorized:false`,
  `InsecureSkipVerify`, trust-all managers) to get past an outage.
- **Don't restore removed trust silently.** If a legitimate service breaks, raise
  an exception ([workbook *Exceptions* sheet](../eMudhra-response-workbook.xlsx)).
- **Test both routes**: the emSign route and, on Windows, Java and older or
  vendor bundles, the Sectigo AAA cross-sign route.
- **A fix on disk is not a fix** until the running process has restarted and a
  fresh connection has been tested.

## Relationship to the response pack

These playbooks reorganise and extend the
[eMudhraResponse pack](../README.md). Closure
routes A/B/C, the six tracker fields and the status values come from the pack's
[action plan](../ACTION_PLAN.md)
and [tracker](../COMPLETION_TRACKER.md).
The deeper caveats are in
[REMEDIATION_OPTIONS](../REMEDIATION_OPTIONS.md)
and [APPLICATION_VALIDATION](../APPLICATION_VALIDATION.md).

None of the commands here have been run against the enterprise estate. Paths
such as `/path/to/...` and `/opt/company/trust/...` are placeholders. Check each
step against the exact deployed version and the current vendor advisory.
