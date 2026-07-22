# Network Sniffing

> Capturing and inspecting network traffic to observe protocols on the wire — and why plaintext services such as classic FTP are unsafe on untrusted networks.

## Overview

**Network sniffing** is the practice of capturing packets as they traverse a network segment and analysing their contents. It is a core skill for both defenders (troubleshooting, monitoring, detection) and attackers (harvesting credentials and data sent without encryption).

In the context of file services, sniffing demonstrates a critical lesson: **plain FTP transmits usernames, passwords, and file data as cleartext**, so anyone able to observe the traffic can read them.

> [!WARNING]
> Only capture traffic on networks and systems you own or are explicitly authorised to test. Unauthorised interception of communications is illegal in most jurisdictions.

## How It Works

- **Wired switched networks** forward frames only to the destination port, so an attacker typically needs port mirroring (SPAN), a tap, or a man-in-the-middle technique (e.g. ARP spoofing) to see other hosts' traffic.
- **Monitor/promiscuous mode** lets an interface accept frames not addressed to it.
- **Wireless networks** in monitor mode can capture frames in radio range subject to encryption.

## Common Tools

| Tool | Use |
|------|-----|
| **Wireshark** | GUI capture and deep protocol analysis |
| **tcpdump** | Lightweight command-line capture |
| **tshark** | Terminal Wireshark, good for scripting |

## Lab: Observing FTP Credentials in Cleartext

1. Start a capture on the interface facing the FTP server.

   ```bash
   sudo tcpdump -i eth0 -w ftp.pcap tcp port 21
   ```

2. From a client, authenticate to the FTP server and transfer a file.
3. Stop the capture and open `ftp.pcap` in Wireshark.
4. Apply the display filter `ftp` and inspect the `USER` and `PASS` commands — the credentials appear in plaintext.
5. In Wireshark, use **Follow > TCP Stream** to reconstruct the full session.

## Defensive Takeaways

- Replace plain FTP with **FTPS** (FTP over TLS) or **SFTP** (SSH File Transfer Protocol) so credentials and data are encrypted in transit.
- Segment and monitor networks; restrict who can mirror or tap traffic.
- Use switch security features (port security, DHCP snooping, dynamic ARP inspection) to limit man-in-the-middle attacks.

## References

- Wireshark User Guide — <https://www.wireshark.org/docs/>
- tcpdump manual — <https://www.tcpdump.org/manpages/tcpdump.1.html>

## Related

- **Secure-FTP-FTPS-and-SFTP**
- [File-Transfers](File-Transfers.md)
