# NETSH Command

`netsh` (Network Shell) is the built-in Windows command-line scripting utility for displaying and modifying the network configuration of a running computer. It works through a tree of **contexts** (interface, advfirewall, wlan, http, portproxy, and more), each exposing its own sub-commands for one area of networking.

## Overview

`netsh` can be run interactively — you enter a context and issue commands within it — or as a single one-line command that names the full context path. It configures IP addressing, firewall rules, wireless profiles, port proxies, and can export/import an entire configuration. Many `netsh` areas overlap with modern PowerShell cmdlets (`Net*` modules), but `netsh` remains the fastest scriptable interface on every supported Windows version.

Common contexts:

| Context | Purpose |
| --- | --- |
| `interface` (`int`) | IPv4/IPv6 addressing, DNS, interface state |
| `advfirewall` | Windows Defender Firewall rules and profiles |
| `wlan` | Wireless profiles, connections, and diagnostics |
| `portproxy` | TCP port forwarding / proxying |
| `http` | URL ACLs and SSL certificate bindings |
| `winsock` | Winsock catalog reset |

> [!NOTE]
> Most configuration changes (firewall rules, IP addresses, resets) require an elevated (Administrator) command prompt. Read-only `show` commands generally do not.

## Syntax

```cmd
netsh [context] [sub-context] command [parameters]
```

Interactive form (prompt changes to show the context):

```text
netsh> interface ip
netsh interface ip> show config
```

## Parameters

| Parameter / verb | Description |
| --- | --- |
| `show` | Display current configuration for the context |
| `set` | Change an existing configuration item |
| `add` | Create a new item (rule, address, profile, proxy) |
| `delete` | Remove an item |
| `dump` | Emit the context's configuration as a re-runnable script |
| `name=` | Interface or rule name to target |
| `source=static \| dhcp` | Addressing mode for `interface ip set address` |
| `dir=in \| out` | Firewall rule direction |
| `action=allow \| block` | Firewall rule action |
| `protocol= / localport=` | Firewall rule protocol and port(s) |
| `/?` | Context-sensitive help at any level |

## Examples

### Interface / IP configuration

Show all interface configuration:

```cmd
netsh interface ip show config
```

Set a static IPv4 address, mask, and gateway:

```cmd
netsh interface ip set address name="Ethernet" source=static addr=192.168.1.50 mask=255.255.255.0 gateway=192.168.1.1
```

Switch an interface back to DHCP:

```cmd
netsh interface ip set address name="Ethernet" source=dhcp
```

Set a static and a secondary DNS server:

```cmd
netsh interface ip set dns name="Ethernet" source=static addr=8.8.8.8
netsh interface ip add dns name="Ethernet" addr=1.1.1.1 index=2
```

### Windows Defender Firewall (advfirewall)

Show the firewall state per profile:

```cmd
netsh advfirewall show allprofiles
```

Add an inbound allow rule for RDP:

```cmd
netsh advfirewall firewall add rule name="Allow RDP" dir=in action=allow protocol=TCP localport=3389
```

Block outbound traffic to a port:

```cmd
netsh advfirewall firewall add rule name="Block 445 Out" dir=out action=block protocol=TCP remoteport=445
```

Turn all firewall profiles off / on (lab or troubleshooting only):

```cmd
netsh advfirewall set allprofiles state off
netsh advfirewall set allprofiles state on
```

Delete a rule by name:

```cmd
netsh advfirewall firewall delete rule name="Allow RDP"
```

### Wireless (wlan)

List saved wireless profiles:

```cmd
netsh wlan show profiles
```

Reveal the stored key for a profile in cleartext:

```cmd
netsh wlan show profile name="MyWiFi" key=clear
```

Export/import a wireless profile (key stored, not shown):

```cmd
netsh wlan export profile name="MyWiFi" folder=C:\wifi key=clear
netsh wlan add profile filename="C:\wifi\Wi-Fi-MyWiFi.xml"
```

### Port proxy (TCP forwarding)

Forward local port 8080 to an internal host — useful for pivoting/relaying:

```cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=10.0.0.5
netsh interface portproxy show all
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
```

### Backup / restore configuration

Dump a context to a script and re-apply it:

```cmd
netsh advfirewall firewall dump > firewall_rules.txt
netsh dump > full_netsh_config.txt
```

Reset the TCP/IP stack and Winsock (repairs corrupt stacks; requires reboot):

```cmd
netsh int ip reset
netsh winsock reset
```

## Enterprise Usage

- **Deployment scripting**: push static IP/DNS or firewall baselines through startup scripts, SCCM/Intune, or imaging pipelines.
- **Firewall management at scale**: `netsh advfirewall firewall add/set rule` is scriptable on every host; GPO-managed firewall policy is preferred for domain-wide enforcement.
- **Diagnostics**: `netsh wlan show wlanreport` and `netsh int ip show config` gather standardized network state for support tickets.
- **Migration/backup**: `netsh dump` captures a complete, re-runnable snapshot of network configuration before a change window.

## Security Considerations

> [!WARNING]
> `netsh wlan show profile name="<SSID>" key=clear` prints saved Wi-Fi passwords in plaintext to any user in the Administrators context — a common credential-harvesting step on compromised laptops.

- `netsh interface portproxy` creates covert TCP relays used for pivoting and lateral movement; audit unexpected portproxy entries.
- `netsh advfirewall set allprofiles state off` disables host firewalling entirely — a high-signal event that should alert.
- Firewall rules added via `netsh` persist across reboots; attacker-created allow rules are a persistence/backdoor technique.
- Monitor for firewall-rule and portproxy changes (Windows Filtering Platform events, event 2004/2006 in the firewall log).

## Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| `The requested operation requires elevation.` | Not running as Administrator | Reopen the command prompt elevated |
| `The filename, directory name, or volume label syntax is incorrect.` | Interface `name=` mismatch (localized/renamed adapter) | Confirm the exact name with `netsh interface show interface` |
| Firewall rule added but traffic still blocked | Another higher-priority block rule or profile mismatch | Review `netsh advfirewall firewall show rule name=all` and the active profile |
| Wi-Fi `key=clear` shows nothing | Profile stored without a key or open network | Verify the profile exists with `netsh wlan show profiles` |
| Network broken after `netsh int ip reset` | Reset is pending | Reboot to complete the TCP/IP stack reset |

## References

- <https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh>
- <https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts>
- <https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/use-netsh-to-reset-tcpip>

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Windows-Firewall-and-AV-Commands](Windows-Firewall-and-AV-Commands.md) — Defender Firewall and AV command reference — related note
- [Network-Enumeration](Network-Enumeration.md) — host and network reconnaissance from the CLI — related note
- Windows-Privilege-Escalation — portproxy pivoting and firewall abuse for privilege escalation — related note
