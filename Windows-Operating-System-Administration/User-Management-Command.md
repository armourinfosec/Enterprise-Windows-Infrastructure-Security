# Windows User Management Commands

Manage local and domain user accounts, groups, privileges, and permissions using built-in Windows command-line tools.

> [!NOTE]
> Most commands require an elevated Command Prompt or PowerShell (Run as Administrator).

---

## Overview

Windows exposes user, group, and permission administration through a small set of built-in command-line utilities that ship on every host. They are the fastest way to inspect and change accounts without a GUI, which is exactly why they are central to routine administration, to privilege escalation, and to attacker persistence in Active Directory environments. The same tasks are covered graphically in [User-Management](User-Management.md) and with cmdlets in [PowerShell-User-Group-Management](PowerShell-User-Group-Management.md); granting local admin rights specifically is detailed in [Add-User-to-Administrators](Add-User-to-Administrators.md), the built-in Administrator account and SIDs in [Windows-Local-Administrator-Account-and-SID](Windows-Local-Administrator-Account-and-SID.md), and the log entries these commands generate are read in [Windows-Event-Logs](Windows-Event-Logs.md).

Windows provides several built-in utilities for user and group administration.

| Command | Description |
| -------- | ----------- |
| `whoami` | Display information about the current user |
| `net user` | Create, modify, and manage user accounts |
| `net localgroup` | Manage local security groups |
| `net group` | Manage Active Directory groups |
| `takeown` | Take ownership of files and folders |
| `icacls` | View and modify NTFS permissions |
| `gpresult` | Display applied Group Policy |
| `nltest` | Query Active Directory information |
| `query session` | View Remote Desktop sessions |
| `logoff` | Log off Remote Desktop sessions |

---

## Viewing User Privileges

### Display the Current User

```cmd
whoami
```

### Display Complete User Information

```cmd
whoami /all
```

Displays:

- User SID
- Group memberships
- Assigned privileges
- Logon information

### Display Group Memberships

```cmd
whoami /groups
```

### Display Assigned Privileges

```cmd
whoami /priv
```

---

## Viewing Users and Groups

### List Local Users

```cmd
net user
```

### Display Information About a User

```cmd
net user Armour
```

```cmd
net user u1
```

```cmd
net user p1
```

Display information about the current user.

```cmd
net user %USERNAME%
```

### List Local Groups

```cmd
net localgroup
```

### View the Local Administrators Group

```cmd
net localgroup administrators
```

### View Other Local Groups

```cmd
net localgroup users
```

```cmd
net localgroup "Remote Desktop Users"
```

```cmd
net localgroup "Remote Management Users"
```

---

## Managing Users

### Create a New User

Without a password:

```cmd
net user armour /add
```

With a password:

```cmd
net user u1 @rmour123 /add
```

| Parameter | Description |
| ---------- | ----------- |
| `u1` | Username |
| `@rmour123` | Password |

### Add a User to the Administrators Group

```cmd
net localgroup administrators armour /add
```

### Delete a User

```cmd
net user u1 /delete
```

> [!TIP]
> `/delete` and `/del` are equivalent.

### Change a User Password

```cmd
net user armour 123456789
```

### Force Password Change at Next Logon

```cmd
net user armour /logonpasswordchg:yes
```

### Configure an Account to Never Expire

```cmd
net user armour /expires:never
```

---

## Enabling and Disabling Accounts

### Enable an Account

Enable the built-in Administrator account.

```cmd
net user administrator /active:yes
```

Enable another account.

```cmd
net user armour /active:yes
```

### Disable an Account

```cmd
net user administrator /active:no
```

```cmd
net user armour /active:no
```

---

## Managing Group Membership

### Remove a User from the Administrators Group

```cmd
net localgroup administrators armour /delete
```

---

## File Ownership and NTFS Permissions

### Take Ownership of a File

```cmd
takeown /f runme.bat
```

### View File Permissions

```cmd
icacls runme.bat
```

### Grant Full Control

```cmd
icacls runme.bat /grant u1:F
```

### Legacy CACLS Example

```cmd
cacls runme.bat /G Armour:F
```

> [!WARNING]
> `CACLS` is deprecated. Use `ICACLS` for all new administration tasks.

---

## Active Directory (Domain) Management

> [!IMPORTANT]
> The following commands apply to Active Directory environments.

### Create a Domain User

```cmd
net user armour @rmour123 /add
```

### List Domain Groups

```cmd
net group /domain
```

### Add a User to Domain Admins

```cmd
net group "Domain Admins" armour /add /domain
```

```cmd
net group "Domain Admins" u1 /add /domain
```

### Add a User to Enterprise Admins

```cmd
net group "Enterprise Admins" armour /add /domain
```

### Add a User to the Domain Administrators Group

```cmd
net group "Administrators" u1 /add /domain
```

### List Domain Controllers

```cmd
nltest /dclist:armour.local
```

### View Applied Group Policy

Summary:

```cmd
gpresult /R
```

Generate an HTML report:

```cmd
gpresult /H report.html
```

---

## Remote Desktop Administration

### Connect from Linux Using rdesktop

```bash
rdesktop -d armour.com -u u11 -p @rmour123 192.168.1.32
```

### View Active Remote Desktop Sessions

```cmd
query session
```

### Log Off a Remote Session

```cmd
logoff <SessionID>
```

Example:

```cmd
logoff 2
```

---

## Example Workflow

Create a user.

```cmd
net user u1 @rmour123 /add
```

Add the user to the Administrators group.

```cmd
net localgroup administrators u1 /add
```

Verify membership.

```cmd
net localgroup administrators
```

Display the user's group memberships.

```cmd
whoami /groups
```

---

## Security Considerations

Nearly every command on this page is dual-use. `net user /add`, `net localgroup administrators /add`, and `net group "Domain Admins" /add /domain` are core administration commands and, in the same breath, classic attacker techniques for **persistence** and **privilege escalation** — a foothold that quietly adds a rogue local admin or drops a controlled account into a privileged domain group. `takeown` and `icacls` are similarly abused to seize ownership of and rewrite permissions on files an attacker should not be able to touch.

> [!WARNING]
> Adding an account to **Administrators**, **Domain Admins**, or **Enterprise Admins** is one of the highest-impact actions on a Windows network. Treat every such change as security-relevant: log it, alert on it, and review it during incident response.

From the defensive side, these commands leave a trail in the Windows Security log when the corresponding audit subcategories are enabled (see [Windows-Audit-Policy](Windows-Audit-Policy.md) and [Windows-Event-Logs](Windows-Event-Logs.md)):

| Event ID | Meaning |
| -------- | ------- |
| 4720 | A user account was created |
| 4722 | A user account was enabled |
| 4724 | An attempt was made to reset an account's password |
| 4726 | A user account was deleted |
| 4728 | A member was added to a security-enabled global group (e.g. Domain Admins) |
| 4732 | A member was added to a security-enabled local group (e.g. Administrators) |
| 4738 | A user account was changed |

---

## Best Practices

- Follow the principle of least privilege — grant administrative rights only when a task requires them.
- Use strong passwords and enable password expiration where appropriate.
- Audit privileged groups (Administrators, Domain Admins, Enterprise Admins) regularly for unexpected members.
- Disable or remove unused accounts, and prefer **ICACLS** over the deprecated **CACLS** utility.
- Use Group Policy to manage account and permission settings consistently across the enterprise.

---

## Troubleshooting

| Issue | Solution |
| ------ | -------- |
| System error 5 has occurred | Run Command Prompt as Administrator. |
| Access is denied | Verify permissions and administrative privileges. |
| The user name could not be found | Confirm the account exists with `net user`. |
| The specified local group does not exist | Verify the group name using `net localgroup`. |
| Domain commands fail | Ensure the computer is joined to the domain and use `/domain` where required. |

---

## References

- [net user (Windows Commands, Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-user)
- [net localgroup (Windows Commands, Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/net-localgroup)
- [icacls (Windows Commands, Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)
- [takeown (Windows Commands, Microsoft Learn)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/takeown)

---

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md)
- [Net-Services-Suite](../Windows-Commands/Net-Services-Suite.md)
- [Add-User-to-Administrators](Add-User-to-Administrators.md)
- Offensive-Active-Directory
- Privilege-Escalation
- [TAKEOWN](../File-Services-and-DFS/TAKEOWN-Command.md)
- [ICACLS](../File-Services-and-DFS/ICACLS-Command.md)
- [CACLS](../File-Services-and-DFS/CACLS-Command.md)