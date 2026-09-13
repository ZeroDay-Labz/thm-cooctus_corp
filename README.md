# Cooctus Corp: Active Directory Penetration Test Writeup

Full walkthrough and findings report for the **Crocc Crew** room on TryHackMe, an Insane-difficulty Active Directory box built around the fictional `COOCTUS.CORP` domain.

Starting with no credentials, this engagement chains exposed credentials, a weak service-account password, and a Kerberos constrained delegation misconfiguration to escalate from anonymous access all the way to full Domain Administrator and NTDS.dit extraction.

> **Disclaimer:** All activity was performed in an authorized, controlled TryHackMe lab environment for training and validation purposes. Nothing in this repository targets real-world systems.

## Room Details

| Field | Value |
|-------|-------|
| Platform | TryHackMe |
| Room | Crocc Crew |
| Category | Active Directory |
| Difficulty | Insane |
| Domain | COOCTUS.CORP |
| Target | Domain Controller (DC.COOCTUS.CORP) |

## Attack Chain

1. **Reconnaissance** — Nmap identified a Windows DC hosting Microsoft IIS 10.0 alongside core AD services (DNS, Kerberos, LDAP, SMB, RDP, WinRM).
2. **Web Enumeration** — `robots.txt` disclosed a backup file (`db-config.bak`) containing plaintext database credentials.
3. **Initial Access** — An RDP login-screen "sticky note" leaked valid credentials for the `Visitor` account.
4. **Foothold & Enumeration** — `Visitor` creds gave SMB share access (user flag) and authenticated LDAP enumeration.
5. **Kerberoasting** — The `password-reset` service account (registered SPN) was Kerberoasted and cracked offline against `rockyou.txt`.
6. **Privilege Escalation** — `password-reset` held Kerberos Constrained Delegation with Protocol Transition. S4U2Self / S4U2Proxy were abused to forge a service ticket impersonating the Domain Administrator.
7. **Domain Dominance** — A DCSync attack extracted the Administrator NTLM hash, then Pass-the-Hash over WinRM yielded an interactive DA shell on the Domain Controller.

## Techniques Demonstrated

- Web content discovery and sensitive information disclosure
- Pre-auth credential disclosure (RDP)
- SMB / LDAP / RPC enumeration
- Kerberoasting (TGS-REP, etype 23 / RC4-HMAC)
- Kerberos Constrained Delegation abuse (S4U2Self / S4U2Proxy)
- DCSync (DRSUAPI replication)
- Pass-the-Hash

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

The full findings report, including the detailed attack narrative, evidence, CVSS scoring, and remediation guidance, is available in this repository:

- [`Cooctus_Corp_Report_Final.docx`](./Cooctus_Corp_Report_Final.docx)

## Credits

Report structure based on the TCM Security PNPT / PJPT sample report template. All original TCM Security logos and identifying information have been removed.
