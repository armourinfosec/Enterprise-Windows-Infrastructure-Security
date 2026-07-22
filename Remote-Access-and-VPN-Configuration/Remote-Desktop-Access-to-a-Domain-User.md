# Remote Desktop Access to a Domain-User

Remote Desktop Services (RDS) allow users to connect to a computer remotely, accessing its desktop interface and resources as if they were physically present. This note covers granting a domain user RDP access at scale using Group Policy and PowerShell in an Active Directory environment.

## Overview

Group Policy, a feature of Windows Server, enables IT administrators to manage and configure operating systems, applications, and user settings in an Active Directory environment. By leveraging Group Policy, administrators can enable Remote Desktop on multiple computers simultaneously, enforce security settings, and specify which users or groups have remote access permissions. This centralized approach simplifies management and ensures consistency across the network.

> [!IMPORTANT]
> **Prerequisites**
> - **Active Directory environment** — your network is configured with AD, and the target computers are part of the domain.
> - **Administrative privileges** — you have administrative rights to create and modify Group Policy Objects (GPOs).
> - **Windows Firewall configuration** — Remote Desktop requires the appropriate firewall rules enabled to allow incoming Remote Desktop connections.

## GUI Steps

### Step 1: Open Group Policy Management Console (GPMC)

1. Log in to your **Domain Controller** or a workstation with the Group Policy Management feature installed.
2. Press `Win + R`, type `gpmc.msc`, and press **Enter** to launch the Group Policy Management Console.

> [!NOTE]
> **Screenshot**
> ![Group Policy Management Console open, with the domain tree expanded on the left and a target OU selected](placeholder-screenshot)

### Step 2: Create a new Group Policy Object (GPO)

1. In the **GPMC**, navigate to your domain or the specific Organizational Unit (OU) that contains the computers you wish to configure.
2. Right-click on the domain or OU, and select **"Create a GPO in this domain, and Link it here…"**
3. Name the new GPO (e.g. **"Enable Remote Desktop"**) and click **OK**.

### Step 3: Edit the GPO to enable Remote Desktop

1. Right-click on the newly created GPO and select **"Edit"** to open the **Group Policy Management Editor**.
2. Navigate to:

```text
Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Connections.
```

3. In the right pane, locate and double-click **"Allow users to connect remotely using Remote Desktop Services"**.
4. Set the policy to **"Enabled"** and click **Apply**, then **OK**.

### Step 4: Configure Network Level Authentication (recommended)

1. Navigate to:

```text
Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Remote Desktop Services -> Remote Desktop Session Host -> Security.
```

2. Double-click on **"Require user authentication for remote connections by using Network Level Authentication"**.
3. Set the policy to **"Enabled"** to enhance security by requiring users to authenticate before establishing a remote session.

### Step 5: Allow Remote Desktop users

1. Navigate to:

```text
Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Restricted Groups.
```

2. Right-click on **Restricted Groups** and select **"Add Group"**.
3. In the **Group** field, type **"Remote Desktop Users"** and click **OK**.
4. In the properties window, click **Add** under **"This group is a member of"** and add the users or groups you wish to grant Remote Desktop access.
5. Click **Apply** and **OK** to save the settings.

### Step 6: Configure Windows Firewall to allow Remote Desktop connections

1. Navigate to:

```text
Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Windows Defender Firewall with Advanced Security -> Inbound Rules.
```

2. Right-click on **Inbound Rules** and select **"New Rule"**.
3. Choose **"Port"** and click **Next**.
4. Select **"TCP"** and specify **port 3389** (the default port for Remote Desktop).
5. Click **Next**, select **"Allow the connection"**, and proceed through the wizard to apply the rule.

### Step 7: Link the GPO to the appropriate OU

1. If you haven't already linked the GPO to the desired OU, do so by right-clicking on the OU and selecting **"Link an existing GPO"**.
2. Choose the **GPO** you created and click **OK**.

### Step 8: Update Group Policy on target computers

1. To apply the new settings immediately, force a Group Policy update on the target computers.
2. On each target computer, open **Command Prompt** and execute:

```cmd
gpupdate /force
```

3. Alternatively, wait for the regular Group Policy refresh interval, which occurs every **90 minutes** by default.

> [!TIP]
> **Video walkthrough**
> [Video: Group Policy](https://www.youtube.com/watch?v=fwivtXh48lE)

## PowerShell

Automate granting Remote Desktop access to a domain user across the estate.

### Step 1: Add user to Remote Desktop Users group on all computers

Add a domain user (e.g. `DomainUser`) to the **Remote Desktop Users** group on all domain-joined computers.

```powershell
$DomainUser = "YourDomain\YourUser"   # Change this to the domain user you want to add
$Computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

foreach ($Computer in $Computers) {
    Invoke-Command -ComputerName $Computer -ScriptBlock {
        param($User)
        Add-LocalGroupMember -Group "Remote Desktop Users" -Member $User -ErrorAction SilentlyContinue
    } -ArgumentList $DomainUser -Credential (Get-Credential)
}
```

> [!NOTE]
> This script retrieves all domain computers and adds the user to the **Remote Desktop Users** group on each one. If you're running this from a non-admin machine, you might need to specify credentials.

### Step 2: Enable Remote Desktop on all computers

```powershell
$Computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

foreach ($Computer in $Computers) {
    Invoke-Command -ComputerName $Computer -ScriptBlock {
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    } -Credential (Get-Credential)
}
```

This enables RDP and configures the firewall to allow RDP connections.

### Step 3: Grant Remote Desktop logon rights

Allow a user to log in remotely via RDP:

```powershell
$User = "YourDomain\YourUser"
$GPO = "Allow RDP Access"

# Create a new GPO (if it doesn't exist)
New-GPO -Name $GPO -ErrorAction SilentlyContinue

# Grant the user "Allow log on through Remote Desktop Services" right
Set-GPRegistryValue -Name $GPO -Key "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -ValueName "UserAuthentication" -Type DWord -Value 1

# Link GPO to the domain
New-GPLink -Name $GPO -Target "OU=Computers,DC=YourDomain,DC=com"
```

> [!NOTE]
> Change `"OU=Computers,DC=YourDomain,DC=com"` to your actual **OU path**.

### Step 4: Force Group Policy update

Apply the GPO changes across all machines:

```powershell
Get-ADComputer -Filter * | ForEach-Object { Invoke-Command -ComputerName $_.Name -ScriptBlock { gpupdate /force } }
```

## Security Considerations

> [!WARNING]
> - Always enable **Network Level Authentication (NLA)** (Step 4) so authentication happens before a full RDP session is built, removing the pre-auth attack surface.
> - Grant RDP rights to a **dedicated AD security group**, not "any domain user" — scope the `Remote Desktop Users` membership tightly.
> - Do not expose port **3389** directly to the internet; front it with an [Remote-Desktop-Gateway](Remote-Desktop-Gateway.md) (HTTPS/443) or require VPN first.
> - Enforce **account lockout** to blunt password spraying against RDP-facing accounts.

## Troubleshooting

| Symptom | Likely cause | Action |
|---|---|---|
| GPO settings not applied | Refresh not run / wrong OU link | Run `gpupdate /force`; confirm the GPO is linked to the OU holding the computer objects |
| Connection refused on 3389 | Firewall rule missing | Verify the "Remote Desktop" inbound rule / Step 6 firewall port rule |
| User can connect but is denied logon | Not in Remote Desktop Users group | Add the user via Step 5 / the Step 1 PowerShell script |

## References

- [Allow access to your PC — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/remotepc/remote-desktop-allow-access)
- [Video: Group Policy](https://www.youtube.com/watch?v=fwivtXh48lE)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Remote Access and VPN Configuration](../Readme.md) — module hub — related note
- [RDP](RDP.md) — the Remote Desktop Protocol itself — related note
- [Remote-Desktop-Gateway](Remote-Desktop-Gateway.md) — publishing RDP externally over HTTPS — related note
- [Managing-Domain-Users-and-Groups-with-PowerShell](../Active-Directory-Domain-Services-AD-DS/Managing-Domain-Users-and-Groups-with-PowerShell.md) — provisioning the domain users granted RDP — related note
- [Active-Directory-Domain-Services](../Active-Directory-Domain-Services-AD-DS/Active-Directory-Domain-Services.md) — domain context for the RDP access — related note
- [Group-Policy(GPO)](../Group-Policy-Objects-GPO/Group-Policy(GPO).md) — centrally enforcing the RDP and NLA policy — related note
