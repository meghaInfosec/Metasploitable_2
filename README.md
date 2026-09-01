# 🛡️ Metasploitable 2 — Penetration Testing Lab

A hands-on educational penetration testing exercise performed against **Metasploitable 2**, an intentionally vulnerable Linux VM built by Rapid7 for security training purposes.

> ⚠️ **Educational use only.** All testing was performed in an isolated local lab environment (Kali Linux attacker VM ↔ Metasploitable 2 target VM, both on a private `192.168.1.0/24` network) with full authorization, as the tester owns and controls both machines. None of the techniques, commands, or findings here should be used against any system without explicit written authorization.

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| Attacker machine | Kali Linux 2026.1 (VirtualBox VM), IP `192.168.1.80` |
| Target machine | Metasploitable 2, IP `192.168.1.50` |
| Hypervisor | Oracle VirtualBox (Windows host) |
| Network mode | Bridged Adapter — private `192.168.1.0/24` LAN |

---

## 🎯 Objectives

This lab walks through a complete penetration testing workflow:

1. **Lab Setup** — configuring VMs and verifying connectivity
2. **Reconnaissance** — system enumeration post-access
3. **FTP Exploitation** — anonymous login + vsftpd 2.3.4 backdoor
4. **SMB Enumeration** — smbclient + enum4linux + file access
5. **Samba Exploitation** — usermap_script root shell
6. **Reverse Shell Delivery** — msfvenom + raw netcat methods
7. **Web Application Testing** — Gobuster + Nikto + DVWA
8. **Findings Summary** — consolidated vulnerability report

---

## 📂 Repository Structure

```
Metasploitable_2/
│
├── README.md                        ← This file
├── 01-lab-setup.md                  ← VM config, network setup
├── 02-reconnaissance.md             ← System enum, /etc/shadow
├── 03-ftp-exploitation.md           ← Anonymous FTP login
├── 04-vsftpd-backdoor.md            ← vsftpd 2.3.4 backdoor exploit
├── 05-smb-enumeration.md            ← smbclient + enum4linux
├── 06-samba-exploit.md              ← Samba usermap_script root shell
├── 07-reverse-shells.md             ← msfvenom + netcat shells
├── 08-web-application.md            ← Gobuster + Nikto + DVWA
└── 09-findings-summary.md           ← Complete findings + remediation
```

---

## 📊 Summary of Findings

| # | Finding | Severity |
|---|---------|----------|
| 1 | vsftpd 2.3.4 backdoor — root shell via port 6200 | 🔴 Critical |
| 2 | Root-level Meterpreter session obtainable | 🔴 Critical |
| 3 | `/etc/shadow` readable, weak MD5-crypt hashes | 🔴 Critical |
| 4 | Default credentials (`msfadmin:msfadmin`) | 🔴 Critical |
| 5 | Anonymous FTP login allowed | 🔴 Critical |
| 6 | Samba 3.0.20 usermap_script RCE | 🔴 Critical |
| 7 | Anonymous SMB login + read/write access | 🟠 High |
| 8 | 34 user accounts exposed via enum4linux | 🟠 High |
| 9 | No account lockout policy | 🟠 High |
| 10 | Apache 2.2.8 — end-of-life | 🟠 High |
| 11 | PHP 5.2.4 — end-of-life, multiple CVEs | 🟠 High |
| 12 | phpMyAdmin exposed unauthenticated | 🟠 High |
| 13 | phpinfo.php publicly accessible | 🟡 Medium |
| 14 | Directory indexing enabled | 🟡 Medium |
| 15 | Missing security headers (CSP, HSTS, etc.) | 🟢 Low |
| 16 | HTTP TRACE method enabled (XST) | 🟢 Low |

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| `nmap` | Port scanning and service detection |
| `ftp` | FTP client for anonymous login testing |
| `smbclient` | SMB share listing and file access |
| `enum4linux` | Deep SMB/Samba enumeration |
| `msfconsole` | Metasploit Framework — exploitation |
| `msfvenom` | Payload generation |
| `netcat (nc)` | Raw reverse shell listener |
| `gobuster` | Web directory enumeration |
| `nikto` | Automated web vulnerability scanning |
| `ssh` | Remote access using default credentials |

---

## 👤 Author

**Megha** — Senior Patent Analyst & Cybersecurity Student  
PG Certificate in AI/GenAI Powered Cybersecurity  
IIT Roorkee × Futurense (Cohort 2025–26)  
GitHub: [@meghaInfosec](https://github.com/meghaInfosec)
