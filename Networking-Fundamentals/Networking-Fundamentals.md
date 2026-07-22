# Networking Fundamentals

A **network** is a collection of two or more interconnected devices that communicate, share resources, and exchange data. Networking fundamentals are the addressing, protocol, and topology basics that every Windows infrastructure service — and every network attack — depends on.

## Overview

Before configuring DNS, DHCP, or Active Directory, you need to speak the underlying language of the network: how devices are addressed, how far a network reaches, who plays which role, and how the wiring is laid out. This note is the entry point for the module — it frames the vocabulary that the deeper notes expand on, such as [IP-Address](IP-Address.md) and [IP-Address-Versions](IP-Address-Versions.md) for addressing, [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md) for network boundaries, [Network-Protocol](Network-Protocol.md) and [TCP-vs-UDP](TCP-vs-UDP.md) for how data moves, and [The-OSI-Model-and-TCP-IP-Model](The-OSI-Model-and-TCP-IP-Model.md) for the layered reference models that tie it all together.

> [!NOTE]
> **Net + Work**
> The word says it plainly: a network is devices doing *work* together over a shared medium. Everything else — addressing, routing, protocols — exists to make that shared work reliable and unambiguous.

## Types of Networks

Networks are classified primarily by **geographic scale** and whether they are wired or wireless.

| **Network Type** | **Definition** | **Devices Used** | **Range** | **Devices Supported** | **Latest Trends/Technologies** |
| :-- | :-- | :-- | :-- | :-- | :-- |
| **LAN** (Local Area Network) | Connects devices within a small area like homes, offices, or buildings. | Switch, Router | 4m – 100m | 2 to 100 PCs | High-speed Ethernet (e.g., 10GBASE-T), Wi-Fi 6/6E |
| **CAN** (Campus Area Network) | Links multiple LANs within a localized campus like universities or large companies. | Layer 3 Switch | 4m – 1km | 2 to 1000 PCs | Software Defined Networking (SDN) adoption |
| **MAN** (Metropolitan Area Network) | Connects multiple LANs or CANs over a city or metro area. | Router | 4m – city-wide | Multiple LANs | 5G and fiber optics deployment |
| **WAN** (Wide Area Network) | Covers broad geographic locations such as different cities or countries. | Router, Firewall | 4m – worldwide | Multiple networks globally | MPLS, SD-WAN for optimized routing |
| **PAN** (Personal Area Network) | Short-range network for personal devices (phones, wearables, laptops). | Bluetooth, Zigbee, NFC devices | Up to 10m | Few devices | Bluetooth 5.x, UWB technologies |
| **WLAN** (Wireless LAN) | Wireless equivalent of LAN using Wi-Fi standards. | Wi-Fi Router, Access Point | 20m – 50m | 2 to 50 PCs | Wi-Fi 6/6E/7, WPA3 security protocol |
| **WCAN** (Wireless CAN) | Wireless version of CAN across campus areas. | Wi-Fi Access Point (AP) | 20m – 1km | 2 to 250 PCs | Mesh Wi-Fi network deployments |
| **WMAN** (Wireless MAN) | Wireless coverage over metropolitan areas, often with Customer Premises Equipment (CPE). | Wi-Fi CPE, microwave links | 20m – 15km | Multiple points | 5G, Fixed Wireless Access (FWA) |
| **WWAN** (Wireless WAN) | Wireless network over very large areas using cellular or satellite technologies. | Satellites | 15km – 1000s km | Cellular devices, IoT devices | 5G/6G, Low Earth Orbit (LEO) satellite networks |

The wired types nest by scale — a PAN sits inside a LAN, LANs join into a CAN, CANs into a MAN, and MANs into a WAN — with each larger scope relying on more capable routing devices:

```mermaid
graph LR
    PAN[PAN: personal, ~10m] --> LAN[LAN: building, ~100m]
    LAN --> CAN[CAN: campus, ~1km]
    CAN --> MAN[MAN: city-wide]
    MAN --> WAN[WAN: worldwide]
```

## IANA (Internet Assigned Numbers Authority)

IANA plays a **critical role** in the global Internet ecosystem by coordinating key resources:

- **IP Address Allocation:** Manages global IPv4 and IPv6 address pools, delegating blocks to Regional Internet Registries (RIRs).
- **DNS Management:** Oversees the root zone and delegates Top-Level Domains (TLDs) such as `.com`, `.net`.
- **Protocol Number and Port Assignments:** Ensures unique identifiers for Internet protocols, ports, and parameters.
- Operated by **ICANN** (Internet Corporation for Assigned Names and Numbers) since 1998.

## Other Important Network Types

- **Internet:** The global interconnected network infrastructure using Internet Protocol (IP) to enable global communication.
- **Intranet:** A private, internal network restricted to organizations offering secure collaboration and resources.
- **VPN (Virtual Private Network):** Securely extends a private network over public infrastructure, encrypting traffic and ensuring privacy.

## Network Roles

| **Role** | **Definition** |
| :-- | :-- |
| **Server** | Provides resources, data, services, or applications to other devices over the network. |
| **Client** | Accesses services and resources provided by a server. |
| **Peer** | In Peer-to-Peer networks, acts both as server and client, sharing resources equally. |

## Network Categories

- **Peer-to-Peer (P2P):** Decentralized, devices (peers) share resources and data directly without central authority.
- **Server-Based:** Centralized model where one or more servers manage services and resources for clients.

See [Workgroup-vs-Peer-to-Peer-vs-Point-to-Point](Workgroup-vs-Peer-to-Peer-vs-Point-to-Point.md) for how these categories map to real Windows network organization models.

## Network Topologies

A **topology** describes how devices are physically or logically arranged and connected. See [Network-Topology](Network-Topology.md) and [Networking-Devices-and-Transmission-Media](Networking-Devices-and-Transmission-Media.md) for the devices and cabling that realize each layout.

| **Topology** | **Description** | **Advantages** | **Disadvantages** |
| :-- | :-- | :-- | :-- |
| **Bus** | All devices connected to a single communication line (bus). | Simple setup, cost-effective | Limited scalability, single point failure |
| **Star** | Devices connect to a central hub or switch. | Easy management, robust to node failures | Hub failure affects all devices |
| **Ring** | Devices connected in a closed loop; data travels in one direction. | Reduced collisions, orderly flow | Failure in one device can disrupt network |
| **Mesh** | Devices interconnected with multiple redundant links. | High reliability, fault tolerance | High cost and complex wiring |
| **Tree** | Hierarchical combination of star and bus topologies. | Scalable, easy to manage | If root fails, large network impact |
| **Hybrid** | Combines two or more topologies to suit network needs. | Flexible, scalable | Complexity in design and management |

## Recent Trends in Networking

- **Software-Defined Networking (SDN):** Centralizes network control, enabling programmable and dynamic network management.
- **Network Function Virtualization (NFV):** Virtualizes network services traditionally run on hardware.
- **Wi-Fi 7 (802.11be):** Next-gen Wi-Fi standard promising ultra-low latency and higher throughput.
- **5G and Beyond:** High-speed mobile networks enabling massive IoT and low-latency applications.
- **Zero Trust Security:** Network security model assuming no implicit trust; continuous verification required.
- **Internet of Things (IoT):** Expansion of network-connected devices, requiring scalable, secure networks.
- **Edge Computing:** Processing data closer to source devices to reduce latency and bandwidth use.

## Security Considerations

Understanding the fundamentals is the difference between seeing "normal traffic" and spotting an attack. Most LAN-layer protocols assume a cooperative network, and on a hostile segment that assumption *is* the vulnerability.

> [!WARNING]
> **Layer 2 is trust-by-default**
> - **Broadcast/name-resolution spoofing** — legacy name services such as [NetBIOS-Name-Service(NBNS)](NetBIOS-Name-Service(NBNS).md), LLMNR, and mDNS answer freely and can be poisoned (Responder-style attacks) to capture authentication hashes.
> - **MAC addresses are not identity** — a [Media-Access-Control(MAC)-Address](Media-Access-Control(MAC)-Address.md) is trivially spoofed; never treat it as an authentication boundary.
> - **Flat networks amplify blast radius** — a single broadcast domain lets a compromised host reach every peer. Segment with VLANs to shrink attack scope.
> - **Rogue infrastructure** — rogue DHCP servers, ARP spoofing, and DNS tampering all exploit the same trust-by-default posture; knowing normal protocol behavior is how you detect them.

- Weak credentials remain the fastest path in — see [How-Secure-Is-My-Password](How-Secure-Is-My-Password.md) for password-strength intuition.
- In offensive engagements, always operate within scope and lawful authority; [Section-91-of-the-Code-of-Criminal-Procedure-(CrPC)-1973-(India)](Section-91-of-the-Code-of-Criminal-Procedure-(CrPC)-1973-(India).md) covers the lawful-production legal context in India.

## Best Practices

- Plan address space with room to grow and document subnet boundaries before deploying services.
- Prefer switched, segmented networks (VLANs) over flat ones to limit broadcast and attack scope.
- Retire legacy name resolution (NetBIOS/LLMNR) where possible — it is a common credential-theft vector.
- Choose topology for the reliability the site needs: redundant (mesh/hybrid) links for critical paths, simpler star for manageability.
- Baseline normal traffic so anomalies (rogue DHCP, ARP/DNS spoofing) stand out.

## Troubleshooting

| Symptom | Likely cause & fix |
| :-- | :-- |
| Two hosts on the same subnet can't communicate | Mask/gateway mismatch or wrong VLAN — verify the [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md) and that both are in the same broadcast domain |
| Name resolves inconsistently | Legacy NetBIOS/LLMNR racing DNS — disable them and rely on DNS |
| Device gets no address / a `169.254.x.x` address | DHCP unreachable or exhausted scope; check the DHCP server and link before assigning statically |
| Whole segment drops when one device fails | Bus or ring topology single-point failure — move to a star/switched design |

## References

- [RFC 791 — Internet Protocol](https://www.rfc-editor.org/rfc/rfc791)
- [IANA — About us](https://www.iana.org/about)
- [Cloudflare Learning — OSI model](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [The Hacker Recipes — LLMNR/NBT-NS poisoning](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/llmnr-nbtns-mdns-spoofing)

## Related

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
- [The-OSI-Model-and-TCP-IP-Model](The-OSI-Model-and-TCP-IP-Model.md) — layered reference models
- [Network-Protocol](Network-Protocol.md) — protocols that move data
- [TCP-vs-UDP](TCP-vs-UDP.md) — connection-oriented vs connectionless transport
- [IP-Address](IP-Address.md) — IP addressing basics
- [IP-Address-Versions](IP-Address-Versions.md) — IPv4 vs IPv6
- [Network-Mask-Subnet-Mask-Net-Mask](Network-Mask-Subnet-Mask-Net-Mask.md) — subnet masks and network boundaries
- [Rules-for-Assigning-an-IP-Address-to-a-Device](Rules-for-Assigning-an-IP-Address-to-a-Device.md) — valid address-assignment rules
- [Media-Access-Control(MAC)-Address](Media-Access-Control(MAC)-Address.md) — MAC addressing
- [NetBIOS-Name-Service(NBNS)](NetBIOS-Name-Service(NBNS).md) — NetBIOS name service
- [Networking-Devices-and-Transmission-Media](Networking-Devices-and-Transmission-Media.md) — switches, routers, cabling, media
- [Network-Topology](Network-Topology.md) — physical and logical topologies
- [Workgroup-vs-Peer-to-Peer-vs-Point-to-Point](Workgroup-vs-Peer-to-Peer-vs-Point-to-Point.md) — network organization models
- [How-Secure-Is-My-Password](How-Secure-Is-My-Password.md) — password-strength intuition
