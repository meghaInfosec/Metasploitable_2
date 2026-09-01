# 07 — Complete Findings Summary

## Lab Details

| Field | Value |
|-------|-------|
| Date | September 1, 2026 |
| Attacker Machine | Kali Linux 2026.1 (192.168.1.80) |
| Target Machine | Metasploitable2 (192.168.1.50) |
| Lab Type | Controlled home lab — educational purpose only |

---

## Complete Vulnerability Report

### 🔴 Critical Findings

| # | Vulnerability | Service | Port | CVE | Impact |
|---|--------------|---------|------|-----|--------|
| 1 | vsftpd 2.3.4 Backdoor | FTP | 21 | CVE-2011-2523 | Root shell via port 6200 |
| 2 | Samba usermap_script | SMB | 139/445 | CVE-2007-2447 | Root shell via command injection |
| 3 | Anonymous FTP Login | FTP | 21 | - | Unauthenticated file access |
| 4 | Anonymous SMB Login | SMB | 139/445 | - | Unauthenticated share access |
| 5 | 34 Users Exposed | SMB | 139/445 | - | Complete user enumeration |
| 6 | No Account Lockout | SMB | 139/445 | - | Unlimited brute force possible |

---

### 🟠 High Findings

| # | Vulnerability | Details |
|---|--------------|---------|
| 7 | Password Complexity Disabled | Minimum 5 chars, no complexity |
| 8 | tmp Share Read/Write Access | Anyone can read and write files |
| 9 | Samba Version Exposed | 3.0.20-Debian revealed in banner |
| 10 | Multiple Vulnerable Services | MySQL, Postgres, Tomcat, Telnet all running |

---

## Attack Chain Summary

```
RECONNAISSANCE
      ↓
nmap scan → open ports identified
      ↓
ENUMERATION
      ↓
enum4linux → 34 users, Samba 3.0.20, password policy
smbclient → shares listed, tmp accessible
      ↓
EXPLOITATION
      ↓
┌─────────────────────────────────┐
│  Path 1: FTP Anonymous Login    │
│  ftp 192.168.1.50               │
│  username: anonymous            │
│  → File system access           │
└─────────────────────────────────┘
      ↓
┌─────────────────────────────────┐
│  Path 2: vsftpd Backdoor        │
│  Metasploit → use 1             │
│  set RHOSTS + LHOST             │
│  exploit → Meterpreter session  │
│  → ROOT shell ✅                │
└─────────────────────────────────┘
      ↓
┌─────────────────────────────────┐
│  Path 3: Samba Exploit          │
│  Metasploit → usermap_script    │
│  set RHOSTS 192.168.1.50        │
│  exploit → Command shell        │
│  whoami → root ✅               │
└─────────────────────────────────┘
      ↓
POST EXPLOITATION
      ↓
Full root access on Metasploitable2
```

---

## Tools Used

| Tool | Purpose | Command |
|------|---------|---------|
| `ftp` | FTP client | `ftp 192.168.1.50` |
| `smbclient` | SMB share listing + access | `smbclient -L //192.168.1.50` |
| `enum4linux` | Deep SMB enumeration | `enum4linux 192.168.1.50` |
| `msfconsole` | Metasploit Framework | `msfconsole` |
| `nmap` | Port scanning | `nmap -sV 192.168.1.50` |
| `ping` | Connectivity test | `ping 192.168.1.50` |
| `ifconfig` | IP address check | `ifconfig` |

---

## Lessons Learned

### For Attackers (Penetration Testers):
1. Always start with **reconnaissance and enumeration** before exploiting
2. Old software versions = easy targets — always check versions
3. Anonymous access is a goldmine of information
4. Metasploit makes exploitation fast — but understand the underlying concepts first
5. Multiple paths to root often exist on poorly configured systems

### For Defenders (Blue Team):
1. **Patch and update** all software regularly
2. **Disable anonymous access** on FTP and SMB
3. **Enable account lockout** to prevent brute force
4. **Enforce strong password policies** — min 12 chars, complexity required
5. **Minimize version disclosure** — hide software banners
6. **Monitor unusual network activity** — especially outbound connections
7. **Principle of least privilege** — services should not run as root
8. **Verify software integrity** using checksums before installing

---

## Complete Remediation Plan

| Priority | Action | Service |
|----------|--------|---------|
| 🔴 Immediate | Upgrade vsftpd (remove backdoor) | FTP |
| 🔴 Immediate | Upgrade Samba to patched version | SMB |
| 🔴 Immediate | Disable anonymous FTP login | FTP |
| 🔴 Immediate | Disable anonymous SMB access | SMB |
| 🟠 High | Enable account lockout policy | All |
| 🟠 High | Enforce password complexity | All |
| 🟠 High | Restrict SMB to trusted IPs | SMB |
| 🟡 Medium | Hide service version banners | FTP/SMB |
| 🟡 Medium | Implement intrusion detection | Network |
| 🟢 Low | Regular security audits | All |

---

## Conclusion

Metasploitable2 demonstrates how a combination of **outdated software**, **misconfigured services**, and **weak security policies** can allow an attacker to gain **complete root access** to a system using freely available tools — in under 30 minutes.

The key takeaway: **Security is only as strong as its weakest link.** A single unpatched service or misconfigured setting can expose an entire system.

---

*This report was created for educational purposes as part of the IIT Roorkee PG Certificate in AI/GenAI Powered Cybersecurity program.*
