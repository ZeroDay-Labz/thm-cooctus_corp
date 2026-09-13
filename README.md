<img src="https://cdn-images.tryhackme.com/room-icons/d387f5c6b5c2bfd07451dd27c187e185.png" align="right" width="120" alt="Crocc Crew room icon">

# Cooctus Corp: Active Directory Penetration Test Writeup

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-c11111)](https://tryhackme.com/room/crocccrew)
[![Difficulty](https://img.shields.io/badge/Difficulty-Insane-black)](https://tryhackme.com/room/crocccrew)
[![Category](https://img.shields.io/badge/Category-Active%20Directory-1f6feb)](https://tryhackme.com/room/crocccrew)
[![Type](https://img.shields.io/badge/Type-Challenge-6f42c1)](https://tryhackme.com/room/crocccrew)
![Result](https://img.shields.io/badge/Result-Full%20Domain%20Compromise-success)

> _"Crocc Crew has created a backdoor on a Cooctus Corp Domain Controller. We're calling in the experts to find the real back door!"_

Full walkthrough and findings report for the **[Crocc Crew](https://tryhackme.com/room/crocccrew)** room on TryHackMe, an Insane-difficulty Active Directory challenge built around the fictional `COOCTUS.CORP` domain.

Starting with no credentials, this engagement chains exposed credentials, a weak service-account password, and a Kerberos constrained delegation misconfiguration to escalate from anonymous access all the way to full Domain Administrator and NTDS.dit extraction.

> **Disclaimer:** All activity was performed in an authorized, controlled TryHackMe lab environment for training and validation purposes. Nothing in this repository targets real-world systems.

## Room Details

| Field | Value |
|-------|-------|
| Platform | TryHackMe |
| Room | [Crocc Crew](https://tryhackme.com/room/crocccrew) |
| Type | Challenge |
| Category | Active Directory |
| Difficulty | Insane |
| Domain | COOCTUS.CORP |
| Target | Domain Controller (DC.COOCTUS.CORP) |

## Attack Chain

1. **Reconnaissance:** Nmap identified a Windows DC hosting Microsoft IIS 10.0 alongside core AD services (DNS, Kerberos, LDAP, SMB, RDP, WinRM).
2. **Web Enumeration:** `robots.txt` disclosed a backup file (`db-config.bak`) containing plaintext database credentials.
3. **Initial Access:** An RDP login-screen "sticky note" leaked valid credentials for the `Visitor` account.
4. **Foothold & Enumeration:** `Visitor` creds gave SMB share access (user flag) and authenticated LDAP enumeration.
5. **Kerberoasting:** The `password-reset` service account (registered SPN) was Kerberoasted and cracked offline against `rockyou.txt`.
6. **Privilege Escalation:** `password-reset` held Kerberos Constrained Delegation with Protocol Transition. S4U2Self / S4U2Proxy were abused to forge a service ticket impersonating the Domain Administrator.
7. **Domain Dominance:** A DCSync attack extracted the Administrator NTLM hash, then Pass-the-Hash over WinRM yielded an interactive DA shell on the Domain Controller.

## The Backdoor Objective

The room's core task is to find the persistence account "Crocc Crew" planted on the Domain Controller. Once Domain Admin was reached, enumerating the domain user base surfaced `admCroccCrew`, an account that stands out from the legitimate users by its `adm`-prefixed name and direct reference to the threat actor. That account is the real back door the engagement was tasked with finding.

## Techniques Demonstrated

- Web content discovery and sensitive information disclosure
- Pre-auth credential disclosure (RDP)
- SMB / LDAP / RPC enumeration
- Kerberoasting (TGS-REP, etype 23 / RC4-HMAC)
- Kerberos Constrained Delegation abuse (S4U2Self / S4U2Proxy)
- DCSync (DRSUAPI replication)
- Pass-the-Hash

## MITRE ATT&CK Mapping

| Tactic | Technique | ID |
|--------|-----------|-----|
| Discovery | Network Service Discovery | T1046 |
| Credential Access | Unsecured Credentials: Credentials In Files | T1552.001 |
| Initial Access | Valid Accounts | T1078 |
| Credential Access | Steal or Forge Kerberos Tickets: Kerberoasting | T1558.003 |
| Privilege Escalation | Steal or Forge Kerberos Tickets (S4U / delegation abuse) | T1558 |
| Credential Access | OS Credential Dumping: DCSync | T1003.006 |
| Lateral Movement | Use Alternate Authentication Material: Pass the Hash | T1550.002 |
| Persistence | Create Account: Domain Account (planted `admCroccCrew`) | T1136.002 |

## Tooling

`nmap` · `gobuster` · `rdesktop` · `smbclient` / `netexec` · `rpcclient` · `ldapdomaindump` · `impacket` (GetUserSPNs, findDelegation, getST, secretsdump) · `hashcat` · `evil-winrm`

## Findings Summary

| Severity | Finding | Type |
|----------|---------|------|
| CRITICAL | Kerberos Constrained Delegation abuse leading to domain compromise | AD Misconfig |
| HIGH | Plaintext credentials exposed on RDP login screen | Info Disclosure |
| HIGH | Kerberoastable service account with weak password | Weak Policy |
| MODERATE | Sensitive information disclosure via web backup file | Info Disclosure |

## Business Impact

Full domain compromise is the most severe outcome of an internal assessment. An attacker in this position can read, modify, or destroy any data in the environment, create persistent backdoor accounts, disable security controls, and deploy ransomware domain-wide. Because every credential in the directory was recovered, remediation requires more than patching a single host: it requires full credential rotation and correction of the underlying AD misconfigurations.

## Report

The full findings report is available in this repository. It covers the complete attack narrative, per-step evidence, CVSS-scored findings, and prioritized remediation guidance (immediate, short-term, and strategic).

- [`Cooctus_Corp_Report_Final.docx`](./Cooctus_Corp_Report_Final.docx)

## Credits

Report structure based on the TCM Security PNPT / PJPT sample report template. All original TCM Security logos and identifying information have been removed.
