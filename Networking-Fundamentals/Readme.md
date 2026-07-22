# Networking Fundamentals

The addressing, protocol, and topology basics that every Windows infrastructure service — and every network attack — depends on.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

Before configuring DNS, DHCP, or Active Directory, you need to speak the underlying language of the network. This module covers IP addressing (v4/v6, subnet masks, assignment rules), MAC addressing, the OSI and TCP/IP models, TCP vs UDP, network protocols, devices and transmission media, topologies, and the workgroup-vs-domain distinction. It also touches practical security context (password strength, and a note on lawful-interception provisions) so the fundamentals connect to real-world administration.

## Learning Objectives

By the end of this module you will be able to:

- Explain IPv4/IPv6 addressing, subnet masks, and the rules for assigning an address to a device
- Map traffic through the OSI and TCP/IP models and distinguish TCP from UDP
- Compare network topologies, devices/media, and workgroup vs peer-to-peer vs point-to-point models

## Topics Covered

This module contains **17 notes**.

| Note | Topic |
| --- | --- |
| [Networking-Fundamentals](Networking-Fundamentals.md) | Networking overview |
| [The-OSI-Model-and-TCP-IP-Model](The-OSI-Model-and-TCP-IP-Model.md) | OSI and TCP/IP reference models |
| [Network-Protocol](Network-Protocol.md) | What a network protocol is |
| [TCP-vs-UDP](TCP-vs-UDP.md) | Connection-oriented vs connectionless transport |
| [IP-Address](IP-Address.md) | IP addressing basics |
| [IP-Address-Versions](IP-Address-Versions.md) | IPv4 vs IPv6 |
| [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md) | Subnet masks and network boundaries |
| [Rules-for-Assigning-an-IP-Address-to-a-Device](Rules-for-Assigning-an-IP-Address-to-a-Device.md) | Valid address-assignment rules |
| [Media-Access-Control(MAC)-Address](Media-Access-Control(MAC)-Address.md) | MAC addressing |
| [NetBIOS-Name-Service(NBNS)](NetBIOS-Name-Service(NBNS).md) | NetBIOS name service |
| [Networking-Devices-and-Transmission-Media](Networking-Devices-and-Transmission-Media.md) | Switches, routers, cabling, media |
| [Network-Topology](Network-Topology.md) | Physical and logical topologies |
| [Workgroup-vs-Peer-to-Peer-vs-Point-to-Point](Workgroup-vs-Peer-to-Peer-vs-Point-to-Point.md) | Network organization models |
| [How-Secure-Is-My-Password](How-Secure-Is-My-Password.md) | Password-strength intuition |
| [Section-91-of-the-Code-of-Criminal-Procedure-(CrPC)-1973-(India)](Section-91-of-the-Code-of-Criminal-Procedure-(CrPC)-1973-(India).md) | Lawful-production legal context (India) |
| [ARP](ARP.md) | Address Resolution Protocol (ARP) |
| [IPv6](IPv6.md) | IPv6 addressing |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **Practical-Labs** collection. Until then, subnet a `/24` into smaller networks by hand, verify with `ipcalc`, then capture a DHCP or DNS exchange in Wireshark and identify the OSI layer of each field.

## Best Practices

- Plan address space with room to grow and document subnet boundaries before deploying services
- Prefer switched, segmented networks (VLANs) over flat ones to limit broadcast and attack scope
- Retire legacy name resolution (NetBIOS/LLMNR) where possible — it is a common credential-theft vector

## Security Considerations

> [!WARNING]
> **Layer 2 is trust-by-default**
> Most LAN protocols assume a cooperative network; on a hostile segment that assumption is the vulnerability.

- NetBIOS/LLMNR/NBNS responses can be spoofed to capture hashes (Responder-style attacks) — disable them
- MAC addresses are trivially spoofed; don't treat them as an authentication boundary
- Understand normal protocol behavior so you can spot ARP spoofing, rogue DHCP, and DNS tampering

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Two hosts on the same subnet can't communicate | Mask/gateway mismatch or wrong VLAN — verify subnet mask and that both are on the same broadcast domain |
| Name resolves inconsistently | Legacy NetBIOS/LLMNR racing DNS — disable them and rely on DNS |

## References

- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791)
- [Cloudflare Learning — OSI model](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [The Hacker Recipes — LLMNR/NBT-NS poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/llmnr-nbtns-mdns-spoofing)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md) — related module
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Proxy Server Administration](../Proxy-Server-Administration/Readme.md) — related module
