# Tape Storage

Tape storage uses magnetic tape as the recording medium, historically favored for long-term archiving and backup because of its low cost per terabyte, high capacity, durability, and — critically for security — its ability to be stored **offline and air-gapped**.

## Overview

Tape sits at the coldest tier of the storage hierarchy. Where [Types-of-Internal-Disks](Types-of-Internal-Disks.md) and [RAID-(Redundant-Array-of-Independent-Disks)](RAID-(Redundant-Array-of-Independent-Disks).md) optimize for fast, always-online access, tape optimizes for cheap, high-density, sequential capacity that can be physically removed and shelved. A cartridge draws no power when it is not mounted in a drive, and once ejected it is unreachable over the network — the property that makes tape the backbone of ransomware-resilient backup strategies.

Modern enterprises rarely write to tape by hand. Native Windows backup dropped built-in tape support after Windows Server 2003/2008, so tape libraries today are driven by backup suites (Veeam, Veritas Backup Exec, Commvault, NetBackup). On Linux, tape drives are addressed directly as character devices (for example `/dev/nst0`) using `mt` and `tar`.

## Types of Tape Storage

### Linear Tape-Open (LTO)

- The most widely used open-standard tape format in enterprise environments.
- Multiple generations from **LTO-1 to LTO-9** (LTO-9 shipped in 2021), each increasing native capacity and transfer speed. LTO-9 stores **18 TB native** (up to ~45 TB compressed) per cartridge.
- Uses **linear serpentine** recording (data written in parallel tracks running the length of the tape), not helical scan.
- **Advantages:** high capacity, low cost per TB, long shelf life (30+ years under proper conditions).
- From **LTO-4 onward**, drives support built-in **AES-256 hardware encryption**; from **LTO-3 onward**, **WORM** (Write Once Read Many) cartridges are available for immutable, compliance-grade archives.
- Used predominantly for backup, archiving, and disaster recovery.

### Digital Linear Tape (DLT / Super DLT)

- Enterprise-grade format using **linear** recording (as the name implies), later extended as Super DLT (SDLT).
- Offered higher capacity and faster access than the QIC-era formats it succeeded.
- Largely replaced by LTO because of LTO's superior capacity, speed, and cost-efficiency and its multi-vendor open standard.

### Advanced Intelligent Tape (AIT)

- Developed by Sony using **helical scan** recording.
- Emphasized reliability and strong error correction, and included a memory-in-cassette chip for fast indexing.
- Mostly phased out in favor of LTO.

### Other Tape Formats

- **8mm / DAT (DDS):** helical-scan formats adapted from consumer video/audio for small-scale data storage.
- **Travan & QIC:** older linear formats used in small-business and consumer settings; now largely obsolete in enterprises.

> [!NOTE]
> **Recording geometry, at a glance**
> **Linear serpentine** (LTO, DLT) streams data along the length of the tape in parallel tracks — fast, durable, and dominant today. **Helical scan** (AIT, 8mm, DAT) writes short diagonal stripes with a rotating head — higher density historically, but more head/tape wear. Mislabeling DLT as "helical" is a common error; DLT is linear.

## How It Works

A tape drive streams data sequentially; there is no random seek as on a disk. Backups are written as a continuous stream, and restoring a single file means the drive must **spool forward** to that file's position — hence tape's strength for whole-dataset archive/restore and its weakness for granular, frequent access.

On Linux, the non-rewinding device node `/dev/nst0` lets you append multiple archives to one tape, while `/dev/st0` rewinds after each operation.

```bash
# Inspect drive/tape status and position
mt -f /dev/nst0 status

# Rewind the tape
mt -f /dev/nst0 rewind

# Write an archive to tape (sequential stream)
tar -cvf /dev/nst0 /data

# List the contents of the current archive on tape
tar -tvf /dev/nst0

# Space forward past one file mark to the next archive, then restore
mt -f /dev/nst0 fsf 1
tar -xvf /dev/nst0
```

## Use Cases

- **Backup & archiving:** cost-effective storage for backup copies and long-term archival, especially where regulatory retention is required.
- **Disaster recovery:** offsite tape copies keep data available after hardware failure, site loss, or a destructive cyberattack.
- **Cold storage:** ideal for infrequently accessed data that must be retained for years — historical, financial, and legal records.
- **Big data / high volume:** archiving massive datasets when keeping them on disk would be prohibitively expensive.

## Benefits

- **Cost efficiency:** lowest cost per terabyte of the mainstream media tiers.
- **High capacity:** modern cartridges (LTO-9) hold many terabytes each.
- **Longevity:** 30+ year shelf life under controlled temperature and humidity.
- **Energy efficiency:** an ejected cartridge consumes no power and generates no heat.
- **Security:** offline, air-gapped copies are unreachable by network-borne threats such as ransomware.

## Security Considerations

Tape is a double-edged asset: its offline nature is one of the strongest defenses against modern extortion attacks, but a physical cartridge is a portable, self-contained copy of the data that can walk out the door.

```mermaid
flowchart LR
    A[Production data] --> B[Disk backup - online]
    B --> C[Tape copy written]
    C --> D[Cartridge ejected + offsite / vault]
    D -.air-gap.-> E[Unreachable by network / ransomware]
```

> [!WARNING]
> **Offensive & defensive relevance**
> - **Lost or stolen tapes = data breach.** Numerous disclosed breaches originated from unencrypted backup tapes lost in transit. Always enable **hardware encryption** (LTO-4+ AES-256) and manage keys separately from the media.
> - **Offline copies defeat ransomware.** An air-gapped, offline tape cannot be encrypted or deleted by malware that has compromised the live network — it is the immutable last resort in the **3-2-1 backup rule** (3 copies, 2 media types, 1 offsite/offline).
> - **WORM for immutability.** WORM cartridges prevent tampering or deletion of archived records, supporting compliance and anti-repudiation.
> - **Insider and physical risk.** Tapes are small and portable; enforce chain-of-custody, cataloging, and secure transport for offsite rotation.
> - **Data remanence.** Retired tapes must be securely erased (degaussing) or physically destroyed — decommissioned media is a classic dumpster-diving and data-recovery target.

## Best Practices

- Enable **hardware AES-256 encryption** (LTO-4+) on every tape and store encryption keys separately from the cartridges.
- Follow the **3-2-1 rule** and keep at least one **offline, offsite** copy for ransomware resilience.
- Use **WORM** media for compliance archives that must not be altered.
- **Test restores regularly** — an untested backup is not a backup; verify readability across drive generations.
- Track cartridges with a catalog/barcode system and enforce **chain-of-custody** for offsite transport and eventual secure disposal.

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Drive will not read an older cartridge | LTO drives read back only ~2 generations and write ~1; use a matching-generation drive |
| Restore is extremely slow for one file | Tape is sequential — the drive must spool to the file; expect this and stage frequent-access data on disk |
| "Media not WORM" write error | Job targeted a WORM cartridge with a rewrite/append operation; use rewritable media or a new WORM tape |
| Encrypted tape cannot be restored | Encryption key lost or unavailable; keys must be escrowed independently of the media |
| Frequent read/write errors | Worn or contaminated media/heads; clean the drive and retire aging cartridges past their rated passes |

## References

- [LTO Ultrium technology overview (LTO Program)](https://www.lto.org/technology/)
- [Microsoft Learn — Windows Server Backup overview](https://learn.microsoft.com/en-us/windows-server/administration/windows-server-backup/windows-server-backup)
- [NIST SP 800-88 — Guidelines for Media Sanitization](https://csrc.nist.gov/pubs/sp/800/88/r1/final)

## Related

- [Types-of-Internal-Disks](Types-of-Internal-Disks.md) — compare with disk-based storage tiers
- [RAID-(Redundant-Array-of-Independent-Disks)](RAID-(Redundant-Array-of-Independent-Disks).md) — online redundancy vs. offline archival
- [File-System](File-System.md) — broader storage and file-organization context
- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub
