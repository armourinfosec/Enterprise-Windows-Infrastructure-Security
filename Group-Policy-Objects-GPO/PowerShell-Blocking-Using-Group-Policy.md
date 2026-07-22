# PowerShell Blocking Using Group Policy (GPO)

Blocking or restricting PowerShell through Group Policy is a common hardening measure in enterprise environments. This note covers five approaches — from Software Restriction Policies to AppLocker, WDAC, and Defender ASR — plus logging and the bypass techniques that make simple `powershell.exe` blocks insufficient on their own.

## Overview

Blocking PowerShell through Group Policy reduces the risk of:

- Malware execution
- Script-based attacks
- Unauthorized administrative activity
- Post-exploitation frameworks (e.g., PowerShell Empire, Cobalt Strike)

> [!NOTE]
> **Choose the right strictness**
> There are multiple approaches depending on how strict the restriction needs to be. Blocking a single executable path is the weakest; application control (AppLocker/WDAC) combined with logging is the strongest.

## Configuration

### Method 1 — Block powershell.exe Using Software Restriction Policies (SRP)

#### Step 1: Open Group Policy Management

On a Domain Controller:

1. Open:

    ```cmd
    gpmc.msc
    ```

2. Create or edit an existing GPO, for example:

    ```text
    Block PowerShell Policy
    ```

3. Link it to the required OU or domain.

> [!NOTE]
> **Screenshot**
> ![GPMC editor with Software Restriction Policies > Additional Rules selected, showing a new Path Rule dialog set to the powershell.exe path with security level Disallowed](placeholder-screenshot)

#### Step 2: Navigate to Software Restriction Policies

```text
Computer Configuration
└── Windows Settings
    └── Security Settings
        └── Software Restriction Policies
```

If no policies exist, right-click and select:

```text
New Software Restriction Policies
```

#### Step 3: Create Path Rules

Navigate to `Additional Rules` and create new **Path Rules** with security level **Disallowed** for each PowerShell binary.

PowerShell 64-bit:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

PowerShell 32-bit:

```text
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
```

PowerShell ISE:

```text
C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe
```

#### Step 4: Apply the GPO

On client systems:

```cmd
gpupdate /force
```

Result — users attempting to launch PowerShell will receive:

```text
This program is blocked by group policy.
```

### Method 2 — Block PowerShell Using AppLocker

AppLocker is more secure and flexible than SRP.

Requirements:

- Windows Enterprise/Education
- Application Identity service enabled

Navigate:

```text
Computer Configuration
└── Windows Settings
    └── Security Settings
        └── Application Control Policies
            └── AppLocker
```

Under `Executable Rules`, create Deny rules for:

```text
powershell.exe
powershell_ise.exe
pwsh.exe
```

Recommended paths:

```text
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe
%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell_ise.exe
%ProgramFiles%\PowerShell\7\pwsh.exe
```

Enable the `Application Identity` service via GPO (required for AppLocker enforcement):

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── System Services
```

Set:

```text
Application Identity → Automatic
```

### Method 3 — Disable PowerShell Script Execution

Instead of fully blocking PowerShell, restrict script execution.

```text
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Windows PowerShell
```

Enable the policy `Turn on Script Execution` and set to:

```text
Allow only signed scripts
```

or:

```text
Disable script execution
```

### Method 4 — Remove PowerShell Access Using File Permissions

Deny execution permissions on:

```text
C:\Windows\System32\WindowsPowerShell\
```

> [!WARNING]
> **Not recommended**
> Removing execute access on the PowerShell directory may break system operations that depend on PowerShell. Prefer AppLocker/WDAC.

### Method 5 — Block PowerShell via Windows Defender ASR Rules

Microsoft Defender Attack Surface Reduction (ASR) can restrict PowerShell abuse:

- Block Office from creating child processes
- Block credential stealing
- Block obfuscated scripts

## Administration

### PowerShell Logging GPO

Enable module, script block, and transcription logging:

```text
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Windows PowerShell
```

Enable:

```text
Turn on Module Logging
Turn on PowerShell Script Block Logging
Turn on PowerShell Transcription
```

### Verify PowerShell is Blocked

Test that the interpreters fail to launch:

```cmd
powershell
```

```cmd
powershell.exe
```

```cmd
pwsh
```

Check applied policies:

```cmd
gpresult /r
```

```cmd
gpresult /h report.html
```

## Security Considerations

### Common Bypass Techniques to Consider

Blocking only `powershell.exe` is not always sufficient. Attackers may use:

- `pwsh.exe`
- PowerShell DLL hosting
- `MSBuild`
- `rundll32`
- WMI
- WinRM
- Encoded commands
- .NET reflection
- LOLBins

> [!IMPORTANT]
> **Defense in depth**
> For stronger security: use AppLocker or WDAC, enable Constrained Language Mode, enable PowerShell logging, and monitor Event IDs 4103 and 4104.

### Useful Event IDs

| Event ID | Description |
| --- | --- |
| 4103 | Module Logging |
| 4104 | Script Block Logging |
| 4688 | Process Creation |
| 8004 | AppLocker Block Event |

## Best Practices

### Recommended Enterprise Approach

| Control | Recommendation |
| --- | --- |
| Basic environments | SRP |
| Enterprise environments | AppLocker |
| High security | WDAC + ASR |
| Monitoring | PowerShell Logging + SIEM |
| Script control | Signed scripts only |

## References

- Microsoft Learn — AppLocker: <https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/applocker/applocker-overview>
- Microsoft Learn — PowerShell logging: <https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Domain-Based-Group-Policy-Configuration](Domain-Based-Group-Policy-Configuration.md) — the GPO mechanism delivering the block — related note
- [Default-Domain-Policy](Default-Domain-Policy.md) — baseline policy this hardening extends — related note
- [Administrative-Templates](Administrative-Templates.md) — the registry-backed PowerShell logging policies — related note
- [Windows-Event-Logs](../Windows-Operating-System-Administration/Windows-Event-Logs.md) — where PowerShell logging events land — related note
- [Windows-Audit-Policy](../Windows-Operating-System-Administration/Windows-Audit-Policy.md) — Event ID 4688 process-creation auditing — related note
