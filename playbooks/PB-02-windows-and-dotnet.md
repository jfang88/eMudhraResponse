# PB-02 Windows servers, .NET and PowerShell

**Owner:** Windows platform team (store) · application owners (custom callbacks) · **Typical priority:** P1–P3 depending on impact

## Applies if

- A Windows server or service makes outbound TLS calls through Windows
  certificate validation: .NET Framework, .NET on Windows, IIS
  outbound/ARR, PowerShell, WinHTTP/WinINet, SChannel-based software.
- Or a .NET application runs on **Linux or in a container**. Those use
  OpenSSL and the distro bundle: follow [PB-03](PB-03-linux-and-unix-hosts.md)
  for the store, and this playbook for the code checks.

Java, Python, Node or Go processes on Windows use **their own** stores. Use
their playbooks.

## Fix

**Option 1: Microsoft's disallowed-certificate update (preferred).** Microsoft
blocked the intermediates on 20 October. Servers with automatic root and
disallowed updates enabled receive it without a reboot.

```powershell
# Read-only. 1 for DisableRootAutoUpdate, or 0 for EnableDisallowedCertAutoUpdate,
# means this server won't fetch the block itself.
Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\SystemCertificates\AuthRoot' -ErrorAction SilentlyContinue |
  Select-Object DisableRootAutoUpdate, EnableDisallowedCertAutoUpdate
```

**Option 2: distribute the block yourself** (disconnected or hardened servers):

- Group Policy: *Computer Configuration → Windows Settings → Security
  Settings → Public Key Policies → Untrusted Certificates*. Import the eight
  intermediate certificates from the Security/PKI reference (include the AAA
  cross-certificates).
- Or mirror Microsoft's CTLs internally (`certutil -syncWithWU <folder>`) and
  point servers at that location, per
  [Microsoft's guidance](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/configure-trusted-roots-disallowed-certificates).

Distrusting the specific intermediates is preferred over removing roots. It
blocks both the emSign and the AAA route without breaking legitimate Sectigo
chains.

## Code checks (application owners)

The store fix is bypassed by code that overrides validation. Search the
deployed code and configuration for:

- `ServerCertificateValidationCallback` returning `true`, or
  `ServerCertificateCustomValidationCallback` /
  `HttpClientHandler.DangerousAcceptAnyServerCertificateValidator`
- Custom `X509Chain` policies with `X509RevocationMode.NoCheck` combined with
  `CustomTrustStore` or `TrustMode = CustomRootTrust`. These use a private root
  list that the Windows update does not change.
- PowerShell `-SkipCertificateCheck`, or
  `[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`

A text match isn't a finding until you confirm the code path is live (pack
rule).

## Verify

- Test as the **service identity**, from the same host, with the same client
  stack. A test run as a local administrator doesn't prove the service
  account's behaviour.

  ```powershell
  # Windows PowerShell 5.1 uses .NET Framework and Windows validation.
  foreach ($h in 'dv-g3b-emudhra','dv-g3b-aaa-emudhra','dv-g3b-noct-emudhra','dv-g3b-aaa-noct-emudhra') {
    try   { $r = Invoke-WebRequest "https://$h.we-have-a-problem.com/" -UseBasicParsing -TimeoutSec 20
            "FAIL  $h  HTTP $($r.StatusCode)" }
    catch { "CHECK $h  $($_.Exception.Message)" }
  }
  ```

  A `CHECK` line must show a certificate trust or revocation error. A timeout or
  name-resolution error is inconclusive ([PB-00](PB-00-test-and-evidence.md)).
- Recycle the application pool or restart the service, then confirm one
  normal business call works.

## Record

- Platform team: a **Route A assurance record** per server build ("Windows
  Server 2019/2022 with auto-update on", "disconnected build with GPO
  Untrusted Certificates"), with test results and coverage from your patch or
  compliance tooling.
- App owners: reference that record, and confirm there's no custom callback
  or custom trust mode (tracker field 3). Use Route B if there is.

## Watch out for

- **Linux and container .NET** don't use the Windows store.
- Adding certificates to a custom store isn't distrust.
- Some third-party Windows products ship OpenSSL or Java and ignore the
  Windows store (PB-11).

## Escalate when

- Auto-update is off and Group Policy distribution isn't possible within the
  P1 window.
- A permissive validation callback is live on a P1 path. That's a pre-existing
  weakness made exploitable by this incident.

Depth: [REMEDIATION_OPTIONS: Windows and .NET](../REMEDIATION_OPTIONS.md#windows-and-net).
