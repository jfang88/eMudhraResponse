# PB-08 Containers, Kubernetes, serverless and CI/CD

**Owner:** container platform / DevSecOps · application owners for their images · **Typical priority:** P1 for build pipelines and artifact proxies; otherwise by workload impact

## Applies if

- You build or run container images, Helm charts, Kubernetes workloads,
  serverless functions, or CI runners.
- Or you operate an **artifact proxy** (Artifactory, Nexus) or package mirror.
  These are P1 chokepoints: every build that pulls through them inherits their
  trust decision.

## Key point

A container carries **its own copy** of the CA bundle, and runtimes inside it
may carry another (certifi, Node, Java cacerts, a Go binary). Patching the
host does nothing for the image. The fix is **rebuild and redeploy**, and
package scanners may not flag the issue.

## Fix

1. **Base images.** Rebuild base images on the updated `ca-certificates` (for
   example Ubuntu 24.04 `20260816ubuntu0.24.04.1` or later; Debian, Alpine and
   UBI per their advisories). Publish them with a new tag and digest.
2. **Runtime layers.** In each application image, apply the runtime
   playbook: certifi ≥ 2026.10.18 (PB-05), Node flags or a fixed release
   (PB-06), the Java store (PB-04), a Go rebuild with ≥ 1.27.3/1.26.10 (PB-07).
3. **Scratch and distroless images.** These usually `COPY` a CA file from the
   build stage. Rebuild so the copy comes from the updated bundle.
4. **Redeploy** every workload onto the new digests, including CronJobs,
   init containers and sidecars. Check that rollbacks can't restore old
   digests (pin or retire them).
5. **Mounted trust bundles.** If you distribute CAs through ConfigMaps,
   Secrets or a bundle controller (for example cert-manager *trust-manager*
   with default CAs enabled), update the source bundle or controller package,
   and confirm pods pick up the change (many need a restart).
6. **Serverless.** The provider patches the managed runtime, but CA bundles
   **vendored into your package** (certifi, node_modules, a Go binary) are
   yours: rebuild and redeploy.

## CI/CD and supply chain

| Component | Check |
|---|---|
| Self-hosted runners and agents (GitHub, GitLab, Jenkins, Azure DevOps) | Runner image or host bundle updated (PB-03); runtimes on the runner (Java for Jenkins agents, Node for actions) |
| Artifact proxy (Artifactory, Nexus) | Usually Java: fix its JVM trust store (PB-04) **first**. It protects all builds that pull through it |
| Package managers in builds (pip, npm, Maven/Gradle, Go modules, cargo) | Each uses its runtime's store. Route builds through the remediated artifact proxy where possible |
| Image registry mirrors and pull-through caches | Upstream TLS verification uses the mirror's trust store |
| Deployment tooling (Terraform, Helm, kubectl plugins, Ansible) | Go and Python tools: see PB-07 and PB-05 |

## Verify

```bash
# What bundle does the image carry? (adapt the path to your base)
docker run --rm --entrypoint sh IMAGE@sha256:DIGEST -c \
  'ls /etc/ssl/certs 2>/dev/null | grep -i emsign; cat /etc/os-release | head -2'
# Behavioural test from inside the image, using the app's runtime (see PB-00 and your runtime playbook)
```

Test **one representative deployment per materially different image**, not
every replica. Use registry and deployment data to show which workloads run
the remediated digests.

## Record

- Platform team: a **Route A record per base image** (digest, packages, test
  result).
- App teams: reference the base-image record, plus their runtime evidence. The
  coverage evidence is a deployment report mapping workloads to remediated
  digests.

## Watch out for

- Image scanners (Trivy, Grype) report **package** findings. They don't prove
  which CAs a workload trusts (pack CBOM appendix).
- Long-lived pods keep the old bundle until restarted.
- **Removing the root breaks legitimate partners** under it. Check PB-13
  before mass redeploys.
- Multi-stage builds can copy the CA file from an *old* builder image.

## Escalate when

- A P1 workload's image can't be rebuilt (no pipeline, or a vendor image).
- Your artifact proxy's vendor has no remedy for its embedded JVM.

Depth: [APPLICATION_VALIDATION §2: Containers](../APPLICATION_VALIDATION.md#2-discover-hidden-and-overridden-stores).
