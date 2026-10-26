# PB-16 Identity integrations and mutual TLS

**Owner:** IAM / API platform · application owners · **Typical priority:** P1

## Applies if

- An application or API **validates tokens or assertions** from an external
  identity provider (Entra ID, Okta, Auth0, Google, partner IdPs): OIDC
  discovery and JWKS fetches, SAML metadata refresh, OAuth token endpoint
  calls.
- Or a service **accepts client certificates** (mTLS), especially from
  partners or the internet.

## Why these are Tier 1

| Path | What a forged TLS certificate gives an attacker | Network position needed |
|---|---|---|
| OIDC discovery / **JWKS fetch** over HTTPS | Serve their own signing keys, then mint tokens your API accepts | Yes (DNS, routing or proxy) |
| **SAML metadata** auto-refresh | Swap in their own signing certificate, then forge assertions | Yes |
| OAuth **token endpoint** / client-credentials calls | Capture client secrets and refresh tokens | Yes |
| **Inbound mTLS** trusting public roots | Present a forged client certificate directly | **No**. They connect directly |

## Fix

1. **Find the runtime doing the fetch.** JWKS and metadata are usually
   fetched by a library inside your app (MSAL, Spring Security, Microsoft
   Identity Web, `jose`, `python-jose`, `passport`), so its runtime trust
   store decides. Apply the runtime playbook (PB-04 to PB-07) and treat the
   path as **P1**.
2. **Reduce dependence on live fetches during the incident** where the
   library allows it: cache JWKS with alerting on new `kid` values, or use
   static SAML metadata with a reviewed signing certificate instead of
   auto-refresh.
3. **Inbound mTLS**: the client-certificate trust list should hold **only the
   specific CAs you expect** (our private CA, or named partner CAs), never a
   general public-root bundle.
   - nginx: `ssl_client_certificate` / `ssl_trusted_certificate`
   - IIS / HTTP.sys: the CTL store bound to the site
   - Java: the server-side `trustStore`
   - API gateways: the mTLS truststore resource

   Remove emSign ECC Root CA - G3 and AAA Certificate Services if present,
   and replace any public-root bundle with an explicit list.
4. **Authorisation mapping**: make sure client-certificate identity is mapped
   on more than the subject CN (for example the issuer plus serial, or a
   registered certificate).

## Verify

- JWKS/metadata: from the deployed app, fetch the PB-00 test hosts using the
  same HTTP client and settings the identity library uses. Expect rejection.
- mTLS: confirm the trust list's contents (list the CAs and compare
  fingerprints). With Security/PKI, test that a client certificate from an
  unexpected public CA is refused.

## Record

**Route B** per integration: library and runtime, trust store, test result,
JWKS/metadata hardening, mTLS trust-list contents.

## Watch out for

- Identity libraries often build their **own** HTTP client. Fixing the app's
  main client may not fix the JWKS fetch.
- Managed IdPs and SaaS gateways validate on *their* side (PB-11). Your
  responsibility is what your code fetches and accepts.

## Escalate when

- A P1 API validates tokens with a library whose trust store you can't change.
- Any inbound mTLS endpoint trusts a general public-root bundle.
