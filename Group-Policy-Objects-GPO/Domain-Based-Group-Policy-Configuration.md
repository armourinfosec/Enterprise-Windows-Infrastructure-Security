# Domain-Based Group Policy Configuration

A practical, recipe-driven reference for creating, linking, and applying Group Policy Objects (GPOs) in an Active Directory **domain** using the Group Policy Management Console (GPMC). Each recipe lists the exact GPMC navigation path for a common hardening or configuration task.

## Overview

In a domain, GPOs are authored in GPMC (`gpmc.msc`), linked to the domain root or an Organizational Unit (OU), and pushed to clients on the next policy refresh or via `gpupdate /force`. This note collects the most common domain GPO recipes: Control Panel and personalization lockdown, folder redirection, drive/removable-storage restrictions, software deployment, and non-administrator server logon.

## Administration

### Accessing Group Policy Management in a Domain

To create or edit GPOs in a domain environment:

1. Open the **Group Policy Management Console (GPMC)**:

    ```cmd
    gpmc.msc
    ```

2. Create a new GPO or edit an existing one:
    - Right-click the **Domain** or an **Organizational Unit (OU)**.
    - Select:

        ```text
        Create a GPO in this domain, and Link it here...
        ```

3. Apply policy changes immediately:

    ```cmd
    gpupdate /force
    ```

4. Verify applied policies:

    ```cmd
    gpresult /r
    ```

> [!NOTE]
> **Screenshot**
> ![GPMC console with an OU right-clicked, showing the "Create a GPO in this domain, and Link it here..." context menu item](placeholder-screenshot)

## Configuration

### Control Panel Restrictions

#### Prohibit Access to Control Panel and PC Settings

Restrict users from accessing the Control Panel and Settings application.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Prohibit access to Control Panel and PC settings
```

#### Hide "Programs and Features" Page

Prevent users from viewing or uninstalling installed applications.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Programs
                └── Hide "Programs and Features" page
```

#### Hide "Windows Features"

Block access to optional Windows feature management.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Programs
                └── Hide "Windows Features"
```

#### Hide the Programs Control Panel

Hide the entire Programs category from Control Panel.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Programs
                └── Hide the Programs Control Panel
```

### Personalization Restrictions

#### Desktop and Appearance Restrictions

Prevent users from modifying desktop appearance and themes.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Personalization
```

Recommended settings:

```text
Enable screen saver
Prevent changing color and appearance
Prevent changing color scheme
Prevent changing desktop background
Prevent changing desktop icons
Prevent changing mouse pointers
Prevent changing screen saver
Prevent changing sounds
Prevent changing theme
```

#### Disable the Display Control Panel

Prevent access to display settings.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Control Panel
            └── Display
                └── Disable the Display Control Panel
```

#### Configure Desktop Wallpaper

Apply a standardized wallpaper for all domain users.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Desktop
            └── Desktop
                └── Desktop Wallpaper
```

Example wallpaper paths:

```text
\\192.168.1.51\data\nature.jpg
```

```text
C:\data\nature.jpg
```

#### Prohibit Desktop Changes

Prevent modification of desktop settings.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Desktop
            └── Desktop
                └── Prohibit changes
```

### Folder Redirection

Redirect user folders to centralized network storage.

```text
User Configuration
└── Policies
    └── Windows Settings
        └── Folder Redirection
            └── AppData (Roaming)
```

Example network path:

```text
\\192.168.1.45\emp_user\
```

### Storage and Drive Access Restrictions

#### Deny Access to Removable Storage

Block USB drives and removable storage devices.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── System
            └── Removable Storage Access
                └── All Removable Storage classes: Deny all access
```

#### Hide Specified Drives in File Explorer

Hide selected drives from users.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Windows Components
            └── File Explorer
                └── Hide these specified drives in My Computer
```

#### Prevent Access to Drives from My Computer

Block users from opening local drives.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── Windows Components
            └── File Explorer
                └── Prevent access to drives from My Computer
```

#### Prevent Access to Command Prompt

Restrict usage of `cmd.exe`.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── System
            └── Prevent access to the command prompt
```

#### Prevent Access to Registry Editing Tools

Restrict access to Registry Editor.

```text
User Configuration
└── Policies
    └── Administrative Templates
        └── System
            └── Prevent access to registry editing tools
```

## Enterprise Deployment

### Software Deployment via Group Policy

Deploy MSI packages automatically across the domain.

> [!TIP]
> **Dedicated walkthrough**
> This section is a quick reference. See [Software-Deployment-via-GPO](Software-Deployment-via-GPO.md) for the full packaging, assignment vs. publishing, and troubleshooting workflow.

#### Prerequisites

- Active Directory configured
- `.msi` installer package available
- Administrative privileges
- Shared network location accessible to domain computers

#### Step 1 — Create a Shared Folder

1. Create a shared folder:

    ```text
    \\192.168.1.45\data\
    ```

2. Copy MSI packages into the share. Example installers:

    ```text
    \\192.168.1.45\data\GoogleChromeStandaloneEnterprise64.msi
    \\192.168.1.45\data\Firefox Setup 88.0.1.msi
    ```

#### Step 2 — Configure Share Permissions

| Principal | Permission |
| --- | --- |
| Domain Computers | Read & Execute |
| Administrators | Full Control |

#### Step 3 — Create the GPO

1. Open:

    ```cmd
    gpmc.msc
    ```

2. Right-click the target OU.
3. Select:

    ```text
    Create a GPO in this domain, and Link it here...
    ```

4. Name the policy:

    ```text
    Software Deployment
    ```

#### Step 4 — Configure Software Installation

```text
Computer Configuration
└── Policies
    └── Software Settings
        └── Software Installation
```

Procedure:

1. Right-click → **New** → **Package**.
2. Browse to the UNC MSI path:

    ```text
    \\SERVER\share\software.msi
    ```

3. Select deployment method:

    | Method | Description |
    | --- | --- |
    | Assigned | Automatically installs during startup/logon |
    | Published | Available for manual installation |

#### Step 5 — Link the GPO

1. Right-click the target OU.
2. Select:

    ```text
    Link an Existing GPO
    ```

3. Choose the created GPO.

#### Step 6 — Update Group Policy

```cmd
gpupdate /force
```

Or restart the client systems.

### Non-Administrator User Login on Server

Allow specific non-admin users to log on locally or remotely.

#### Allow Local Logon

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── User Rights Assignment
                    └── Allow log on locally
```

#### Allow Remote Desktop Logon

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── User Rights Assignment
                    └── Allow log on through Remote Desktop Services
```

## Administration Commands

### Force Group Policy Update

```cmd
gpupdate /force
```

### View Applied Policies

```cmd
gpresult /r
```

### Generate Detailed HTML GPO Report

```cmd
gpresult /h report.html
```

### Refresh Computer Policies Only

```cmd
gpupdate /target:computer /force
```

### Refresh User Policies Only

```cmd
gpupdate /target:user /force
```

## Security Considerations

### Physical Security

Use hardware protection mechanisms:

- Kensington locks
- Chassis padlocks
- Restricted server room access
- CCTV monitoring

### Insider Threat Prevention Best Practices

- Enforce least privilege access.
- Monitor user activity and network traffic.
- Restrict local administrator access.
- Audit removable media usage.
- Enable centralized logging and SIEM monitoring.
- Use application whitelisting.
- Implement Data Loss Prevention (DLP).

## Best Practices

- Test all GPOs in a lab or staging OU before production deployment.
- Avoid modifying the Default Domain Policy unnecessarily.
- Use separate GPOs for security restrictions, software deployment, user customization, and device control.
- Document all GPO changes.
- Use security filtering and WMI filters carefully.
- Periodically review inactive or conflicting GPOs.

## Troubleshooting

### Verify Deployment

- Check installed applications: `Control Panel → Programs and Features`.
- Verify GPO application: `gpresult /r`.
- Review Event Viewer:

    ```text
    Applications and Services Logs
    └── Microsoft
        └── Windows
            └── GroupPolicy
    ```

### Common Issues

| Issue | Troubleshooting |
| --- | --- |
| MSI not installing | Verify UNC path accessibility |
| GPO not applying | Run `gpresult /r` |
| Access denied | Check share and NTFS permissions |
| Slow deployment | Verify DNS and network connectivity |
| Installation failed | Review Event Viewer logs |

## References

- Microsoft Learn — Group Policy Management Console: <https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791(v=ws.11)>
- Microsoft Learn — `gpresult`: <https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/gpresult>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Group-Policy(GPO)](Group-Policy(GPO).md) — core Group Policy concepts and local editor — related note
- [Default-Domain-Policy](Default-Domain-Policy.md) — baseline GPO this configuration extends — related note
- [PowerShell-Blocking-Using-Group-Policy](PowerShell-Blocking-Using-Group-Policy.md) — example hardening GPO — related note
- [Software-Deployment-via-GPO](Software-Deployment-via-GPO.md) — full MSI deployment walkthrough — related note
- [Managing-Domain-Users-and-Groups-with-PowerShell](../Active-Directory-Domain-Services-AD-DS/Managing-Domain-Users-and-Groups-with-PowerShell.md) — managing the objects GPOs target — related note
