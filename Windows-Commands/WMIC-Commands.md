# WMIC Commands

WMIC (Windows Management Instrumentation Command-line) is the legacy command-line front end to [WMI](Windows-Registry.md), giving administrators and attackers a scriptable way to query and manage almost any facet of a Windows system from a shell.

## Overview

WMIC exposes WMI classes (`Win32_BIOS`, `Win32_OperatingSystem`, `Win32_Service`, and hundreds more) as simple `wmic <alias> get/list/call` commands. From a single CMD prompt you can read hardware and OS inventory, list services and hotfixes, enumerate accounts, and even create processes on remote hosts. Because it ships in-box and speaks WMI over DCOM, it is a favourite living-off-the-land binary for reconnaissance and lateral movement — see [Network-Enumeration](Network-Enumeration.md) for how it fits into host and domain recon, and [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md) for the modern `Get-WmiObject` / `Get-CimInstance` equivalents.

> [!IMPORTANT]
> **WMIC is deprecated**
> Microsoft has **deprecated** `wmic.exe` and no longer installs it by default on **Windows 11** and **Windows Server 2022+** (it is now an optional *Feature on Demand*). Prefer PowerShell's CIM cmdlets (`Get-CimInstance`) for new work; the sections below cover both re-enabling WMIC and its PowerShell replacements.

## Enabling WMIC on Windows 11 / Server 2022

Microsoft removed WMIC from newer Windows builds. You can still add it back manually as a Windows capability.

### Step 1: Run PowerShell as Administrator

- In the search bar, type **PowerShell**.
- Right-click on **Windows PowerShell** and select **Run as administrator**.
- If prompted by User Account Control (UAC), click **Yes** to grant administrative privileges.

### Step 2: Check if the WMIC Capability is Present

Query the online capability list for WMIC:

```powershell
Get-WindowsCapability -Online | Where-Object { $_.Name -like "*WMIC*" }
```

If the output shows `NotPresent`, WMIC is not installed and you can proceed to the next step:

```text
Name  : WMIC~~~~
State : NotPresent
```

### Step 3: Add the WMIC Capability

Add the capability using the Windows Update client. Make sure the system has an active Internet connection or reaches a local WSUS server:

```powershell
Add-WindowsCapability -Online -Name "WMIC~~~~"
```

### Step 4: Verify Successful Installation

On success the output resembles:

```text
Path          :
Online        : True
RestartNeeded : False
```

`RestartNeeded : False` means the WMIC capability was added without requiring a reboot.

## Command Reference

WMIC commands follow the pattern `wmic <alias> <verb> [properties] [/Format:<fmt>]`. `get` returns named properties, `list brief`/`list full` returns preset property sets, and `/Format:list` prints one property per line. Start with the built-in help:

```cmd
wmic /?
```

> [!TIP]
> **Output formatting**
> Append `/Format:list` to almost any query for a readable key/value dump, or `/Format:csv` / `/Format:htable` for structured output you can post-process. `list brief` is the quickest way to preview an alias.

### BIOS Information

Display selected BIOS fields, the serial number, or the manufacturer:

```cmd
wmic bios get name, version, serialnumber
wmic bios get serialnumber
WMIC BIOS Get Manufacturer
WMIC BIOS Get /Format:list
```

### Computer System Information

Retrieve the computer name, model, product/SMBIOS data, boot configuration, and logical disks:

```cmd
wmic csproduct get name
wmic csproduct Get /Format:list
wmic BOOTCONFIG Get /Format:list
WMIC logicaldisk Get /Format:list
WMIC ComputerSystem GET Model
WMIC ComputerSystem Get /Format:list
wmic computersystem get name,systemtype
```

### Network Information

Enumerate network adapters, MAC addresses, and descriptions:

```cmd
wmic nic get macaddress,description
wmic nic get /Format:list
```

### Hardware Information

Read motherboard, memory, partition, and disk-drive details:

```cmd
wmic baseboard get product,Manufacturer,version,serialnumber
wmic COMPUTERSYSTEM get TotalPhysicalMemory
wmic partition get name,size,type
wmic diskdrive get Name, Manufacturer, Model, InterfaceType, MediaType, SerialNumber
```

### Service Management

List services, or pull name/display name/path/start mode for each — useful for spotting misconfigured or exploitable services during Windows-Privilege-Escalation:

```cmd
wmic service list brief
wmic service get name,displayname,pathname,startmode
```

### Operating System Information

Retrieve install date, local time, and OS caption/build/architecture:

```cmd
WMIC OS GET InstallDate
WMIC OS GET localdatetime
WMIC OS Get /Format:list
WMIC OS Get Caption, BuildNumber, OSArchitecture
```

### Updates and Patches

List installed updates and hotfixes — the patch level often reveals missing fixes to target:

```cmd
wmic qfe get Caption,Description,HotFixID,InstalledOn
wmic qfe Get /Format:list
```

### User and Group Management

Enumerate local user accounts and groups:

```cmd
wmic USERACCOUNT Get /Format:list
wmic GROUP Get /Format:list
```

### Software and Product Information

List installed products (MSI-registered software):

```cmd
wmic PRODUCT Get /Format:list
```

### Disk Drive Information

Show detailed disk-drive information:

```cmd
wmic DISKDRIVE Get /Format:list
```

## PowerShell Alternatives to WMIC

Because WMIC is deprecated, the same data is available through WMI/CIM cmdlets. Prefer `Get-CimInstance` over `Get-WmiObject` on modern systems (`Get-WmiObject` is itself deprecated):

```powershell
# BIOS information
Get-WmiObject Win32_BIOS
Get-CimInstance -ClassName Win32_BIOS

# Computer system information
Get-WmiObject Win32_ComputerSystem
Get-CimInstance -ClassName Win32_ComputerSystem

# Disk drive details
Get-WmiObject Win32_DiskDrive
Get-CimInstance -ClassName Win32_DiskDrive

# Installed updates
Get-HotFix
```

## Security Considerations

WMIC is a classic **living-off-the-land binary (LOLBin)**: it is signed, in-box, and rarely blocked, so attackers use it for both reconnaissance and remote code execution without dropping new tooling. See the [Windows Commands](Readme.md) hub for the broader LOLBin picture and Offensive-Active-Directory for how WMI-based execution is used across a domain.

> [!WARNING]
> **Remote execution and lateral movement**
> With valid credentials, WMIC can spawn a process on a remote host over DCOM (RPC), a common lateral-movement technique:
> ```cmd
> wmic /node:<target-ip> /user:<DOMAIN\user> /password:<pass> process call create "cmd.exe /c <command>"
> ```
> The same `process call create` verb runs locally for execution and can be abused for persistence. Defensively: baseline and log `wmic.exe` execution, monitor WMI activity (Sysmon Event IDs 19/20/21 for WMI subscriptions), restrict who holds administrative rights, and segment the RPC/DCOM ports (135 + dynamic RPC) that remote WMI relies on.

- Treat any interactive `wmic /node:` invocation as high-signal — legitimate admin scripting usually uses PowerShell remoting or CIM sessions instead.
- WMI event-subscription persistence (via `__EventFilter` / `CommandLineEventConsumer`) is a stealthy technique — audit WMI namespaces for unexpected permanent subscriptions.
- Because WMIC exposes account, patch, and service data, it is a fast enumeration step post-compromise — the same queries defenders should baseline are the ones attackers run first.

## Best Practices

- Prefer **PowerShell CIM cmdlets** (`Get-CimInstance`) for new automation; reserve WMIC for constrained shells where PowerShell is unavailable.
- Do not re-enable WMIC on production hosts unless a specific legacy tool requires it — removing it reduces LOLBin surface.
- Log and baseline `wmic.exe` process creation and WMI activity so that anomalous `/node:` or `process call create` usage stands out.
- Use least-privilege service accounts and enforce network segmentation to limit remote WMI (DCOM/RPC) reachability.
- Combine WMIC/`net`/`sc` recon queries into documented baselines so both admins and defenders know what "normal" looks like.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| `'wmic' is not recognized as an internal or external command` | WMIC not installed (Windows 11 / Server 2022+) — add it with `Add-WindowsCapability -Online -Name "WMIC~~~~"` or use `Get-CimInstance` instead |
| `Access is denied` / `Win32: Access is denied` | Not running elevated — reopen CMD/PowerShell as Administrator (UAC) |
| Remote `wmic /node:` hangs or fails | RPC/DCOM (port 135 + dynamic RPC) blocked by firewall, or credentials lack remote admin rights |
| `Invalid query` / no output for an alias | Wrong property or alias name — run `wmic <alias> get /?` to list valid properties |
| `Add-WindowsCapability` fails to download | No Internet / WSUS reachability — connect the host or point it at a valid update source |

## References

- [WMIC command (deprecated) — Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmic)
- [Windows Management Instrumentation — Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page)
- [Get-CimInstance — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance)
- [LOLBAS — Wmic.exe](https://lolbas-project.github.io/lolbas/Binaries/Wmic/)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [PowerShell-Commands-for-Penetration-Testing](PowerShell-Commands-for-Penetration-Testing.md) — related note (`Get-WmiObject` / CIM equivalents in PowerShell)
- [Network-Enumeration](Network-Enumeration.md) — related note (WMIC supplements host and network reconnaissance)
- [Service-Controller-Utility-Commands](Service-Controller-Utility-Commands.md) — related note (`sc` for service management from the CLI)
- [Net-Services-Suite](Net-Services-Suite.md) — related note (the `net` command family)
- Windows-Privilege-Escalation — related note (WMIC to find patches and misconfigurations for privesc)
- Offensive-Active-Directory — related note (WMI-based remote execution and domain recon)
