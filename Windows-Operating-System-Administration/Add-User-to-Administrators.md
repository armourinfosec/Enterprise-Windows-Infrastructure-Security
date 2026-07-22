# Add User to Administrators

## Overview
**Adding a user to the local Administrators group** grants that account full administrative control over a Windows host. This note collects the command-line (`net`) and PowerShell recipes for creating an account, promoting it to Administrators (and Remote Desktop Users), enabling RDP, and toggling the firewall — the exact steps an administrator uses for provisioning and an attacker abuses for persistence and lateral access.

> [!WARNING]
> Disabling the firewall and enabling RDP widen the attack surface. Perform these steps only in a lab or with an explicit change-control justification.

## Commands
### 1. Create a User (`Admintest`)
```cmd
net user Admintest password@123 /add
```

This creates the local account `Admintest`.

### 2. Add the User to the Administrators Group
```cmd
net localgroup Administrators Admintest /add
```

This grants `Admintest` administrative privileges.

### Add the User to the Remote Desktop Users Group
```cmd
net localgroup "Remote Desktop Users" Admintest /add
```

### 3. Enable Remote Desktop
Enable RDP by clearing the `fDenyTSConnections` registry value:

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

Setting `fDenyTSConnections` to `0` enables Remote Desktop. Then allow RDP through the firewall:

```cmd
netsh advfirewall firewall set rule group="Remote Desktop" new enable=yes
```

### 4. Turn Off Windows Firewall
Disable the firewall for all profiles (domain, private, and public):

```cmd
netsh advfirewall set allprofiles state off
```

## Examples
### PowerShell Alternative
Create a user `Admintest2`:

```powershell
New-LocalUser -Name "Admintest2" -Password (ConvertTo-SecureString "password@123" -AsPlainText -Force) -FullName "Admin Test User" -Description "Administrator Test Account"
```

Add `Admintest2` to the Administrators group:

```powershell
Add-LocalGroupMember -Group "Administrators" -Member "Admintest2"
```

Enable Remote Desktop:

```powershell
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0
```

Allow Remote Desktop through Windows Firewall:

```powershell
Get-NetFirewallRule | Where-Object DisplayName -like "*Remote Desktop*" | Enable-NetFirewallRule
```

Turn off Windows Firewall for all profiles:

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

## Best Practices
- Keep the Administrators group as small as possible; audit its membership regularly.
- Prefer per-user RDP access via the Remote Desktop Users group over granting full admin rights.
- Leave the Windows Firewall enabled; scope specific rules instead of disabling it wholesale.
- Never hardcode passwords in scripts — prompt for credentials or use a secure vault.

## Security Considerations
- Adding an account to Administrators is a classic **privilege-escalation** and **persistence** objective; unexpected membership changes should raise alerts.
- Group-membership changes are logged (Event ID 4732) — see [Windows-Event-Logs](Windows-Event-Logs.md) and [Windows-Audit-Policy](Windows-Audit-Policy.md) for capturing them.
- A newly created local admin with RDP enabled and the firewall off is a strong indicator of compromise.

## Troubleshooting
| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| System error 5 has occurred | Command Prompt not elevated | Run as Administrator |
| The user name could not be found | Account does not exist | Create it first with `net user <name> /add` |
| RDP still refuses connections | Firewall rule not applied | Re-run the `netsh advfirewall` rule enable command |
| Password rejected on creation | Password fails complexity policy | Use a compliant password or relax the local policy |

## References
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-localgroup>
- <https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/add-localgroupmember>
- <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netsh-advfirewall>

## Related
- [User-Management-Command](User-Management-Command.md) — the `net user` / `net localgroup` commands used to add the account
- [PowerShell-User-Group-Management](PowerShell-User-Group-Management.md) — PowerShell equivalents for account and group management
- [Windows-Local-Administrator-Account-and-SID](Windows-Local-Administrator-Account-and-SID.md) — the built-in admin account and RID 500
- [Windows-Event-Logs](Windows-Event-Logs.md) — group-membership changes appear as Event ID 4732
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
