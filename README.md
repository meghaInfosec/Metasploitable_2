# 🛡️ Metasploitable2 Penetration Testing Lab

> **Course:** PG Certificate in AI/GenAI Powered Cybersecurity  
> **Institution:** IIT Roorkee × Futurense (Cohort 2025–26)  
> **Lab Environment:** Kali Linux + Metasploitable2 (VirtualBox)

---

## 📌 Lab Overview

This project documents a complete penetration testing exercise performed on an intentionally vulnerable machine (Metasploitable2) using Kali Linux as the attacker machine — entirely within a controlled home lab environment for educational purposes.

---

## 🖥️ Lab Environment

| Machine | OS | IP Address | Role |
|---------|-----|-----------|------|
| Kali Linux | Kali 2026.1 | 192.168.1.80 | Attacker |
| Metasploitable2 | Ubuntu (Metasploitable) | 192.168.1.50 | Target (Victim) |

Both VMs are connected via **Bridged Adapter** on VirtualBox — same LAN, no internet required for attacks.

---

## 📂 Project Structure

```
metasploitable-lab/
│
├── README.md                   ← You are here
├── 01-lab-setup.md             ← VM setup and network configuration
├── 02-ftp-exploitation.md      ← Anonymous FTP login vulnerability
├── 03-vsftpd-backdoor.md       ← vsftpd 2.3.4 backdoor exploit
├── 04-smb-enumeration.md       ← SMB enumeration with smbclient + enum4linux
├── 05-samba-exploit.md         ← Samba usermap_script exploit (root shell)
├── 06-web-application.md       ← DVWA web application testing
└── 07-findings-summary.md      ← Complete findings and recommendations
```

---

## 🎯 Attacks Performed

| # | Attack | Tool Used | Result |
|---|--------|-----------|--------|
| 1 | Anonymous FTP Login | ftp client | ✅ Got in without password |
| 2 | vsftpd 2.3.4 Backdoor | Metasploit | ✅ Meterpreter root shell |
| 3 | SMB Enumeration | smbclient, enum4linux | ✅ 34 users, shares, policy |
| 4 | SMB File Upload | smbclient | ✅ Write access confirmed |
| 5 | Samba Exploit | Metasploit | ✅ Root shell obtained |
| 6 | Web App Access | Firefox browser | ✅ DVWA accessed |

---

## ⚠️ Disclaimer

This lab is performed **strictly for educational purposes** in a controlled, isolated environment. All attacks are performed on intentionally vulnerable software (Metasploitable2). Never perform these techniques on systems without explicit written permission.
