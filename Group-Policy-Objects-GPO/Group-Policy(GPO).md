# Group Policy (GPO)

Group Policy is a feature in Microsoft Windows that allows administrators to centrally manage and configure operating system settings for users and computers within an Active Directory (AD) environment. It helps enforce security policies, software installations, and user restrictions across multiple devices in an organization.

## Overview

A **Group Policy Object (GPO)** is a collection of settings that administrators link to Active Directory containers (Sites, Domains, and Organizational Units) to control the working environment of user accounts and computer accounts. GPOs are the primary mechanism for enforcing a consistent security and configuration baseline across a Windows fleet.

> [!NOTE]
> **Where GPO fits**
> Local Group Policy manages a single machine; Active Directory Group Policy manages users and computers across an entire domain from the Group Policy Management Console (GPMC).

### Key Features of Group Policy

- **Centralized Management:** Manage settings from a central console via Group Policy Management Console (GPMC) or Local Group Policy Editor (`gpedit.msc`).
- **Security Enforcement:** Enforce password policies, restrict access to system settings, disable USB ports, etc.
- **Software Deployment:** Install, update, or remove applications via Group Policy Objects (GPOs).
- **User & Computer Configuration:** Manage settings like desktop restrictions, network settings, and folder redirection.
- **Automation:** Policies apply automatically at startup, shutdown, logon, or logoff, ensuring consistency.

## Concepts

### Types of Group Policy

| Type | Description |
| --- | --- |
| Local Group Policy | Applies to a single computer via `gpedit.msc`. |
| Active Directory Group Policy | Applies to users and computers in an AD domain via GPMC. |

### How Group Policy Works

- **Group Policy Objects (GPOs):** Collections of settings linked to AD objects (Sites, Domains, OUs).
- **Scope Levels:**
    - Local
    - Site
    - Domain
    - Organizational Unit (OU)

## Architecture

Group Policy is processed in a fixed order known as **LSDOU** — Local, Site, Domain, OU — with the last policy applied winning any conflict.

```mermaid
flowchart LR
    A[Local Policy] --> B[Site]
    B --> C[Domain]
    C --> D[Organizational Unit]
    D --> E[Effective Settings<br/>last applied wins]
```

- Processing Order:

    ```text
    Local → Site → Domain → OU (last applied wins)
    ```

> [!TIP]
> **Precedence**
> Because the OU-level policy is applied last, it overrides conflicting settings from the Domain, Site, and Local levels — unless a higher-level GPO is set to **Enforced**. See [GPO-Processing-Order](GPO-Processing-Order.md) for the full precedence rules.

### Policy Refresh

- Automatic every 90–120 minutes (background refresh, with a random offset).
- Manual update:

    ```cmd
    gpupdate /force
    ```

## Administration

### Common Use Cases

- Disable Control Panel / Settings
- Enforce password complexity
- Disable USB storage devices
- Deploy software like Chrome, Firefox
- Redirect folders like Documents to a network share
- Restrict CMD, PowerShell, and Registry access

### Accessing Group Policy

- Open Local Group Policy Editor:

    ```cmd
    gpedit.msc
    ```

- Open Group Policy Management Console (domain):

    ```cmd
    gpmc.msc
    ```

- Force update Group Policy:

    ```cmd
    gpupdate /force
    ```

> [!NOTE]
> **Screenshot**
> ![Local Group Policy Editor (gpedit.msc) open, showing the Computer Configuration and User Configuration trees in the left navigation pane](placeholder-screenshot)

## Configuration

### Local Group Policy Editor

#### Computer Configuration

Affects the entire machine.

- **Software Settings** — Configure software deployment for the whole computer.
- **Windows Settings**
    - Security Settings (Passwords, account lockout, firewall)
    - Startup/Shutdown Scripts
    - Name Resolution Policy
- **Administrative Templates** — Registry-based policies for:
    - System
    - Network
    - Printers
    - Windows Components (Edge, OneDrive, etc.)

#### User Configuration

Affects specific user accounts.

- **Software Settings** — Manage software policies at the user level.
- **Windows Settings**
    - Logon/Logoff Scripts
    - Folder Redirection
- **Administrative Templates** — Controls user settings:
    - Start Menu
    - Control Panel
    - Desktop
    - System restrictions (CMD, Registry)

## Examples

### Control Panel Restrictions

| Action | Path |
| --- | --- |
| Block Control Panel | `User Configuration → Administrative Templates → Control Panel → Prohibit access to Control Panel and PC settings` |
| Hide "Programs and Features" | `User Configuration → Administrative Templates → Control Panel → Programs → Hide Programs and Features page` |
| Disable "Windows Features" | `User Configuration → Administrative Templates → Control Panel → Programs → Hide Windows Features` |
| Hide Programs Control Panel | `User Configuration → Administrative Templates → Control Panel → Programs → Hide the Programs Control Panel` |

### Personalization Restrictions

| Action | Path |
| --- | --- |
| Force specific wallpaper | `User Configuration → Administrative Templates → Desktop → Desktop → Desktop Wallpaper` |
| Disable changing desktop background | `User Configuration → Administrative Templates → Control Panel → Personalization → Prevent changing desktop background` |
| Disable screen saver change | `User Configuration → Administrative Templates → Control Panel → Personalization → Prevent changing screen saver` |
| Prevent theme change | `User Configuration → Administrative Templates → Control Panel → Personalization → Prevent changing theme` |

Example Wallpaper Paths (Force specific wallpaper):

```text
\\192.168.1.51\data\Wallpaper.jpg
```

```text
C:\Users\Public\Pictures\Wallpaper.png
```

### Folder Redirection

Folder Redirection is a feature in Windows Active Directory environments that allows administrators to redirect the path of a user's known folders (such as Documents, Desktop, Downloads, Pictures, etc.) from the local machine to a network location (e.g., a file server). This ensures centralized storage, easier backups, and a seamless user experience across multiple devices.

#### Purpose

- Store user data on a server rather than locally.
- Enable roaming profiles without increasing profile size.
- Centralize data for backup and security.
- Provide a consistent experience across different computers.

#### How It Works

Administrators configure Folder Redirection via Group Policy:

1. Group Policy Management Console (GPMC) → Create/Modify a GPO.
2. Navigate to:

    ```text
    User Configuration > Policies > Windows Settings > Folder Redirection
    ```

3. Right-click a folder (e.g., Documents) → Properties.
4. Choose a redirection setting (e.g., Basic or Advanced).
5. Enter the network path (e.g., `\\FileServer\Users\%USERNAME%\Documents`).

> [!NOTE]
> **Screenshot**
> ![Folder Redirection properties dialog for the Documents folder, showing the Basic redirection setting and the target UNC root path field](placeholder-screenshot)

#### Common Redirectable Folders

- Desktop
- Documents
- Downloads
- Pictures
- Music
- Videos
- Favorites
- Start Menu
- AppData (Roaming)

#### Redirection Options

| Option | Description |
| --- | --- |
| Basic | All users are redirected to the same location format. |
| Advanced | Redirect users based on group membership. |
| Redirect to the following location | Static path like `\\server\share\%USERNAME%` |
| Create a folder for each user under the root path | Automatically makes per-user folders. |

#### Folder Redirection Path

| Action | Path |
| --- | --- |
| Redirect AppData, Desktop, Documents | `User Configuration → Windows Settings → Folder Redirection` |

Example Path:

```text
\\192.168.1.45\emp_user\
```

### Software Deployment via Group Policy

| Action | Path |
| --- | --- |
| Deploy MSI packages | `User Configuration → Software Settings → Software installation → New Package` |

Deployment Methods:

- **Published** → Optional for users (Add/Remove Programs)
- **Assigned** → Mandatory (installs automatically)

Example MSI Paths:

```text
\\192.168.1.45\data\GoogleChromeStandaloneEnterprise64.msi
\\192.168.1.45\data\FirefoxSetup.msi
```

> [!TIP]
> **Full walkthrough**
> See [Software-Deployment-via-GPO](Software-Deployment-via-GPO.md) for the complete MSI packaging, assignment vs publishing, and troubleshooting workflow.

### Storage and Drive Access Restrictions

| Action | Path |
| --- | --- |
| Block all removable storage (USB) | `User Configuration → Administrative Templates → System → Removable Storage Access → All Removable Storage classes: Deny all access` |
| Hide specific drives in Explorer | `User Configuration → Administrative Templates → Windows Components → File Explorer → Hide these specified drives in My Computer` |
| Prevent access to drives | `User Configuration → Administrative Templates → Windows Components → File Explorer → Prevent access to drives from My Computer` |

### Security Restrictions

| Action | Path |
| --- | --- |
| Disable Command Prompt (CMD) | `User Configuration → Administrative Templates → System → Prevent access to the command prompt` |
| Disable Registry Editor (regedit) | `User Configuration → Administrative Templates → System → Prevent access to registry editing tools` |

### Allow Non-Administrator Logins on Servers

| Action | Path |
| --- | --- |
| Allow local logon | `Computer Configuration → Windows Settings → Security Settings → Local Policies → User Rights Assignment → Allow log on locally` |
| Allow Remote Desktop logon | `Computer Configuration → Windows Settings → Security Settings → Local Policies → User Rights Assignment → Allow log on through Remote Desktop Services` |

## Security Considerations

- Folder redirection applies only when users log into domain-joined machines.
- Avoid redirecting folders to offline shares unless Offline Files is enabled.
- Folder redirection can conflict with OneDrive Known Folder Move (KFM) if not configured properly.
- Use NTFS permissions carefully on redirected shares:
    - Users: Full control on their own redirected folder.
    - No permission to other users' folders.
- Use Access-Based Enumeration on the share so users see only their own folders.
- Prevent users from moving files back to local storage.

### Additional Physical & Digital Security

- Use physical security: padlocks, BIOS passwords, hardware locks.
- Monitor insider threats — see [10-Common-Ways-Users-Leak-Data](10-Common-Ways-Users-Leak-Data.md).

## Best Practices

- Keep the Default Domain Policy for account/password/Kerberos settings only; create dedicated GPOs for everything else (see [Default-Domain-Policy](Default-Domain-Policy.md)).
- Test all GPOs in a staging OU before production.
- Document every GPO change and use descriptive GPO names.
- Use security filtering and WMI filters to scope GPOs precisely (see [WMI-Filters](WMI-Filters.md)).
- Periodically review inactive or conflicting GPOs.

## Troubleshooting

| Symptom | Resolution |
| --- | --- |
| Policy not applying | Run `gpupdate /force`, then verify with `gpresult /r` |
| Wrong setting winning | Check LSDOU precedence and Enforced/Block Inheritance ([GPO-Processing-Order](GPO-Processing-Order.md)) |
| User settings ignored | Confirm loopback processing if applied on a server/kiosk ([Loopback-Processing](Loopback-Processing.md)) |

See [GPO-Troubleshooting](GPO-Troubleshooting.md) for the full diagnostic workflow.

## References

- Microsoft Learn — Group Policy overview: <https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy>
- [10 Ways Users Steal Company Data](https://www.proofpoint.com/us/blog/insider-threat-management/10-ways-users-steal-company-data-and-how-stop-them)
- [PC Security Tips (YouTube)](https://www.youtube.com/watch?v=fV_OaldJEUU)
- [Server Hardening Guide (YouTube)](https://www.youtube.com/watch?v=oLOYCL0twR8)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Default-Domain-Policy](Default-Domain-Policy.md) — the baseline domain-wide GPO — related note
- [Domain-Based-Group-Policy-Configuration](Domain-Based-Group-Policy-Configuration.md) — hands-on GPMC configuration recipes — related note
- [GPO-Processing-Order](GPO-Processing-Order.md) — LSDOU precedence, inheritance, and enforcement — related note
- [Software-Deployment-via-GPO](Software-Deployment-via-GPO.md) — deploying MSI applications through GPO — related note
- [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) — the directory GPOs are linked into — related note
- [Windows-Registry](../Windows-Commands/Windows-Registry.md) — Administrative Templates are registry-backed policies — related note
