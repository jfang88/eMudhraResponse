# Team guide: eMudhra response (start here)

**Version 2 · 26 October 2026 · For application owners, developers, IT platform and support teams**

This is the one document to send to delivery teams. It tells you what has
happened, what to do, what to return and by when, and where to find the
steps for your technology.

## What has happened

Eight eMudhra intermediate CA keys were weak from generation in April 2025
and can be recovered by anyone. With them, an attacker can forge TLS
certificates (six keys) or Windows code signatures (two keys) that
unprotected software accepts. **Browsers and OS trust stores were protected
by their vendors on 17–20 October. Software that keeps its own trust list
was not:** Java, Python, Node, Go and Rust applications, containers, build
pipelines, appliances and agents. Your job is to find out which of your
services are in that group, fix them, and record the result.

Separately, Chrome stops trusting **emSign ECC Root CA - G3** on **Tuesday 27
October 2026**. Any of our certificates under that root must be replaced
([PB-13](PB-13-our-certificates-and-availability.md)).

## What you need to do

1. **Find your playbook** in the tables below, by team or by what you run.
2. **Run its *Applies if* test** for each service you own. If nothing
   applies, record Route C with the reason.
3. **Set the priority.** Impact (H/M/L) × verifier susceptibility (A–D or
   Unknown), using the table below. Work P1 first.
4. **Fix** using the playbook's options, in order of preference. Prefer the
   smallest supported change; a full application upgrade is rarely needed.
5. **Verify** the fix in the running workload with
   [PB-00 Test and evidence](PB-00-test-and-evidence.md).
6. **Return one six-field record** per service in the incident tracker (or
   the workbook's *Service tracker* sheet):

   | # | Field | What to provide |
   |---|---|---|
   | 1 | Scope and owner | Service or asset group, accountable owner, environment |
   | 2 | Priority and rationale | Impact, susceptibility, priority, one sentence why |
   | 3 | Trust configuration | Standard platform and its assurance record ID, or the custom trust or uncertainty |
   | 4 | Status and action | Route A, B or C; status; what you changed, or why no change is needed |
   | 5 | Evidence reference | Link to the change, test or configuration record, or the platform assurance record |
   | 6 | Outstanding work | Issue, owner and target date, or "None" |

   **Closure routes.** **A:** covered by a platform team's assurance record,
   with no custom trust overrides. **B:** your own fix and test (custom
   bundles, embedded runtimes, critical paths). **C:** not applicable to the
   assessed role, with evidence. "We don't buy eMudhra certificates" is not a
   reason. Full definitions are in the
   [action plan](../ACTION_PLAN.md#three-closure-routes) and
   [completion tracker](../COMPLETION_TRACKER.md).

## When

| Priority | Target window (indicative, pending decision D-03) |
|---|---|
| **P1** | Contain within 72 hours (by Thu 29 Oct for existing work); close or except within 7 days (by Mon 2 Nov) |
| **P2** | Close or except within 14 days (by Mon 9 Nov) |
| **P3** | Close within 30 days (by Wed 25 Nov) |
| **P4** | Confirm in the normal cycle |

Windows run from when a record is discovered or assigned. If a date agreed
through change control differs, record why. **Escalate suspected exploitation
or a critical outage immediately**, whatever the priority.

## Set the priority

| Impact ↓ / Susceptibility → | A: vendor-pushed block or CT enforced | B: patched OS store, confirm | C: own or compiled bundle | D: embedded or vendor-only | Unknown |
|---|---|---|---|---|---|
| **H** | P3 | **P1** | **P1** | **P1** + exception | **P1 to assess** |
| **M** | P4 | P2 | P2 | P2 + vendor tracking | P2 to assess |
| **L** | P4 | P3 | P3 | P3 + vendor tracking | P3 to assess |

**H** means a forged certificate could expose credentials or sensitive data,
enable privileged actions or code execution, or affect many downstream
systems. Treat these paths as P1 unless the verifier is confirmed safe:
inspection proxies, identity integrations (JWKS, SAML metadata, token
endpoints), build pipelines and artifact proxies, payment and customer-data
APIs, inbound mTLS, code signing and application control, admin and developer
tooling. Definitions and worked cases are in
[Prioritisation](../RISK_TIERS.md).

## Find your playbook by team

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

## Find your playbook by what you run

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

## How each playbook is organised

Every playbook uses the same headings:

1. **Applies if**: a yes/no test.
2. **Fix**: options in order of preference. Use the first one that works.
3. **Verify**: prove the fix is active in the running workload.
4. **Record**: what goes into the six fields.
5. **Watch out for**: the mistakes we expect.
6. **Escalate when**: when to stop and call Security/PKI or the incident lead.

Each playbook ends with **References**. Version-aware detail is in
[Remediation options](../REMEDIATION_OPTIONS.md) and
[Application validation](../APPLICATION_VALIDATION.md).

## Rules that apply everywhere

- **Deleting a cached intermediate is not distrust.** A server can send it again.
- **Adding certificates never removes trust.** For example, `NODE_EXTRA_CA_CERTS`
  only adds.
- **Never turn off verification** (`verify=False`, `-k`, `rejectUnauthorized:false`,
  `InsecureSkipVerify`, trust-all managers) to get past an outage.
- **Don't restore removed trust silently.** If a legitimate service breaks,
  raise an exception (workbook *Exceptions* sheet).
- **Test the AAA route where AAA is still trusted** (Windows, Java, older or
  vendor bundles), as well as the emSign route.
- **A fix on disk is not a fix** until the running process has restarted and a
  fresh connection has been tested.

## Escalate immediately

- You find a certificate or signature chaining to an affected intermediate
  outside the test hosts (a possible forgery: call the SOC).
- A P1 path can't be fixed or contained within 72 hours.
- A fix breaks a business-critical integration, and the only workaround would
  restore removed trust.

None of the commands in these playbooks have been run against the enterprise
estate. Paths such as `/path/to/...` and `/opt/company/trust/...` are
placeholders. Check each step against the exact deployed version and the
current vendor advisory.

## References

**Related documents**

- [README](../README.md): executive summary and situation as at 26 October
- [Action plan](../ACTION_PLAN.md): closure routes, responsibilities, the 30-day plan, decisions
- [Prioritisation](../RISK_TIERS.md): impact, susceptibility and worked cases
- [Completion tracker](../COMPLETION_TRACKER.md): the six fields and status values
- [Remediation options](../REMEDIATION_OPTIONS.md), [Application validation](../APPLICATION_VALIDATION.md), [CBOM and cross-signing](../CBOM_AND_CROSS_SIGNING.md)

**External sources**

1. Incident reference: [Impact, maintained package list and test fixtures](https://emudhra.we-have-a-problem.com/)
2. Google Security Blog: [Protecting Chrome and Android users from compromised eMudhra intermediate CA certificates](https://blog.google/security/protecting-chrome-and-android-users-from-compromised-emudhra-intermediate-ca-certificates/)
3. Cloudflare: [8 eMudhra certificates are compromised. Here's what you need to know](https://blog.cloudflare.com/emudhra-certificate-compromise/)
