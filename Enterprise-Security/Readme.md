# Enterprise Security

The hardening capstone — taking every role built earlier in the course and defending it as one coherent, attack-resistant enterprise.

> [!NOTE]
> **Module hub**
> Part of the **[Enterprise Windows Infrastructure Security](../Readme.md)** course.

## Overview

The earlier modules build a working Windows enterprise; this one hardens it. It covers security baselines, the tiered administration model, credential protection (LAPS, Credential Guard, Protected Users), attack-surface reduction, PKI/AD CS, Kerberos and NTLM hardening, and how the monitoring stack ties it together. The framing is adversary-aware: each control is presented against the specific attack it blunts, so the module doubles as a defender's and an attacker's map of the enterprise.

## Learning Objectives

By the end of this module you will be able to:

- Apply security baselines and a tiered administration model to a Windows/AD environment
- Deploy credential-protection controls (LAPS, Credential Guard, Protected Users) and reduce attack surface
- Harden AD authentication (Kerberos/NTLM) and AD CS, and map controls to the attacks they mitigate

## Topics Covered

This module contains **8 notes**.

| Note | Topic |
| --- | --- |
| [Security-Baselines](Security-Baselines.md) | Microsoft Security Baselines / SCT and CIS Benchmarks |
| [Tiered-Administration-Model](Tiered-Administration-Model.md) | Tier 0/1/2 and Privileged Access Workstations |
| [LAPS](LAPS.md) | Local Administrator Password Solution |
| [Credential-Guard-and-Protected-Users](Credential-Guard-and-Protected-Users.md) | Protecting secrets in memory and privileged accounts |
| [Attack-Surface-Reduction](Attack-Surface-Reduction.md) | ASR rules, AppLocker/WDAC application control |
| [AD-CS-Security](AD-CS-Security.md) | PKI hardening and ESC misconfiguration classes |
| [Kerberos-and-NTLM-Hardening](Kerberos-and-NTLM-Hardening.md) | Delegation, roasting defenses, NTLM restriction |
| [Credential-Theft-Defenses](Credential-Theft-Defenses.md) | Mitigating Mimikatz/pass-the-hash/pass-the-ticket |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided labs for this topic are being consolidated under the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection. Planned exercises include applying a security baseline GPO, deploying LAPS, restricting NTLM, and demonstrating an ASR/AppLocker rule stopping a download cradle.

## Best Practices

- Adopt a tiered admin model and Privileged Access Workstations before adding point controls
- Deploy LAPS, restrict local admin, and enable Credential Guard to break credential-reuse chains
- Baseline with Microsoft/CIS benchmarks, then measure drift continuously rather than one-time

## Security Considerations

> [!WARNING]
> **Defense-in-depth, not a checklist**
> No single control saves an enterprise; attackers chain misconfigurations. Assume breach and layer controls so one failure isn't total.

- AD CS, Kerberos delegation, and NTLM are recurring privilege-escalation avenues — audit them specifically
- Hardening without monitoring is blind — pair every control with detection (see the Monitoring module)
- Test controls adversarially (purple-team) rather than trusting that a GPO "applied"

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| A hardening GPO breaks a legitimate app | Over-broad control (AppLocker/ASR/NTLM restriction) — scope with audit mode first, then enforce |
| LAPS password not rotating | Schema/permission or GPO misconfig — verify the LAPS schema extension and computer self-write rights |

## References

- [Microsoft Security Compliance Toolkit / baselines](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10)
- [Securing privileged access / tiered model](https://learn.microsoft.com/en-us/security/privileged-access-workstations/overview)
- [CIS Microsoft Windows Benchmarks](https://www.cisecurity.org/cis-benchmarks)

## Related Notes

- [Enterprise Windows Infrastructure Security](../Readme.md) — course hub and map of content
- [Active Directory Domain Services (AD DS)](../Active-Directory-Domain-Services-AD-DS/Readme.md) — related module
- [Group Policy Objects (GPO)](../Group-Policy-Objects-GPO/Readme.md) — related module
- [Windows Monitoring and Logging](../Windows-Monitoring-and-Logging/Readme.md) — related module
