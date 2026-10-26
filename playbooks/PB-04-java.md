# PB-04 Java applications and Java-based products

**Owner:** application owner (app trust config) · platform team (shared JDK builds) · **Typical priority:** P1–P2 (susceptibility C: own trust store)

## Applies if

Any JVM makes outbound TLS calls, or accepts client certificates. That
includes app servers (Tomcat, WebLogic, JBoss/WildFly, WebSphere), Kafka
clients, Elasticsearch, Jenkins, **Artifactory and Nexus** (P1 chokepoints),
Atlassian products, desktop Java apps and IDEs.

Java does **not** use the Windows or macOS store by default, and it does not
check revocation by default. Updating the OS does not fix Java.

## Find the trust store the JVM actually uses

```bash
# 1. Which java binary does the service run? (check the systemd unit / service wrapper)
ps -eo pid,args | grep [j]ava
# 2. Effective trust-store properties of the running JVM (JDK tools, same user as the JVM):
jcmd <pid> VM.system_properties | grep -i -E 'javax.net.ssl.trustStore|java.home'
# 3. List candidate entries in the effective store (JDK 9+: -cacerts; JDK 8: -keystore $JAVA_HOME/jre/lib/security/cacerts)
/path/to/runtime/bin/keytool -list -v -cacerts | grep -i -E 'Alias name|Owner:' | grep -i -B1 -E 'emSign|AAA Certificate Services'
```

If `javax.net.ssl.trustStore` is set, that file is the store. Otherwise, check
for `jssecacerts` first, then `cacerts`, under `java.home`. App servers may
configure outbound trust separately from the inbound HTTPS keystore.

The store password is your managed credential (the JDK default is publicly
documented). Enter it interactively; don't put it on a command line or in a
log.

## Fix: pick the first option that applies

**Option 1: vendor remedy for your distribution.**

- **Oracle JDK (8 and later):** follow Oracle's advisory (updated 23 Oct). It
  requires removing the relevant roots from the effective keystore, or
  enabling revocation checking, *even on listed updated builds*. Oracle's
  documented aliases include `emsigneccrootcag3 [jdk]` and
  `comodoaaaca [jdk]`.
- **Azul Zulu:** apply Azul's replacement blocklist file (`blocked.certs` or
  `blacklisted.certs`) for your exact version and platform, check the
  checksum, and restart every JVM that uses it.
- **Other distributions** (Temurin, Corretto, Red Hat, IBM/Semeru, SAP
  Machine): use that vendor's advisory. **Don't** copy another vendor's
  blocklist file.

**Option 2: managed copy of the store** (when the app supports an explicit
trust store, and to avoid touching a shared JDK):

```bash
cp /path/to/effective/cacerts /opt/company/trust/app-cacerts      # keep the original format
/path/to/runtime/bin/keytool -list -v -keystore /opt/company/trust/app-cacerts -alias 'VERIFIED_ALIAS'   # confirm SHA-256 fingerprint
/path/to/runtime/bin/keytool -delete -keystore /opt/company/trust/app-cacerts -alias 'VERIFIED_ALIAS'
# JVM option, set in the service definition:
#   -Djavax.net.ssl.trustStore=/opt/company/trust/app-cacerts
```

Delete only entries whose fingerprint matches the Security/PKI removal set.
Removing the AAA root affects legitimate Sectigo chains that still depend on
it, so run the positive control.

**Option 3: revocation checking** (Oracle documents
`com.sun.net.ssl.checkRevocation` and `com.sun.security.enableCRLDP`). This
only works if CRL/OCSP endpoints are reachable and failure behaviour is
acceptable. Prefer options 1 or 2 for disconnected apps.

## Code checks

Search the deployed code for trust-all `X509TrustManager` implementations, a
`HostnameVerifier` that always returns true, and custom `SSLContext`
initialisation with an in-code keystore. A custom trust manager ignores
everything above.

## Verify

- Oracle links `CheckCerts.java` (works on JDK 8+; on 8, compile it with
  `javac` first). Run it with the **target runtime** and the service's
  trust-store settings.
- Then call the [PB-00](PB-00-test-and-evidence.md) test hosts **through the
  application's own HTTP client**: expect `PKIX path building failed` or
  `CertificateRevokedException`, not a response.
- Restart all JVMs. Pooled connections and long-lived JVMs keep the old store
  until restart.

## Record

- Shared JDK builds: a **Route A record** per distribution/version/store, with
  coverage from deployment tooling.
- Apps with their own store or a custom trust manager: **Route B** (store path,
  change, test results, positive control).

## Watch out for

- **Debian/Ubuntu** generate `/etc/ssl/certs/java/cacerts` from the OS bundle.
  Canonical documents a synchronisation issue; preserve private enterprise CAs.
- Bundled JREs inside vendor products (Atlassian, Jenkins installers,
  monitoring agents) are separate stores. Ask the vendor (PB-11).
- Images and app-server templates must carry the change, or redeploys undo
  it (PB-08).
- **JDK 6/7 and proprietary app servers**: no universal fix has been verified.
  Seek a vendor hotfix or a validated scoped store; track as an exception.

## Escalate when

- The app uses a custom `SSLContext` you can't reconfigure on a P1 path.
- The vendor product embeds a JRE and the vendor has no remedy (PB-11).

Depth: [REMEDIATION_OPTIONS: Java](../REMEDIATION_OPTIONS.md#java-jdk-8-and-later-plus-older-installations) · [Oracle advisory](https://ops.java/emudhra) · [Azul workaround](https://docs.azul.com/core/emudhra-workaround)
