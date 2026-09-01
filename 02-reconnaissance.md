# 02 — Reconnaissance & System Enumeration

## Overview
Once initial access was established via Meterpreter, system enumeration was performed to confirm privilege level and gather intelligence about the target.

---

## Meterpreter Session Enumeration

```
meterpreter > pwd
/
meterpreter > cd root
meterpreter > getuid
Server username: root
meterpreter > shell
Process 5348 created.
Channel 1 created.
```

✅ Session was already running with **root** privileges — no privilege escalation required.

---

## System Fingerprinting

```bash
whoami       # root
id           # uid=0(root) gid=0(root)
uname -a     # Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
hostname     # metasploitable
```

| Finding | Value |
|---------|-------|
| OS | Ubuntu-based Metasploitable build |
| Kernel | 2.6.24-16-server (2008) — multiple known local privilege escalation CVEs |
| Architecture | 32-bit (i686) |
| User context | root (uid=0) |

---

## Network Information

```bash
ifconfig
```

| Interface | IP Address | Notes |
|-----------|-----------|-------|
| `eth0` | 192.168.1.50 | Primary interface |
| `lo` | 127.0.0.1 | Loopback |

---

## Port Scanning from Kali

```bash
nmap -sV 192.168.1.50
```

Key open ports discovered:

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 |
| 23 | Telnet | Linux telnetd |
| 80 | HTTP | Apache 2.2.8 |
| 139/445 | SMB | Samba 3.0.20 |
| 3306 | MySQL | MySQL 5.0.51a |
| 5432 | PostgreSQL | PostgreSQL DB |
| 8180 | HTTP | Apache Tomcat |

---

## Credential Harvesting — /etc/shadow

With root access confirmed, `/etc/shadow` was dumped:

```bash
cat /etc/shadow
```

Accounts found with active password hashes (**MD5-crypt `$1$` format** — weak and crackable with `john` or `hashcat`):

- `root`
- `sys`
- `klog`
- `msfadmin`
- `postgres`
- `user`
- `service`

> **Note:** MD5-crypt hashes are considered weak by modern standards. SHA-512 or bcrypt should be used instead.

---

## SSH Access with Default Credentials

```bash
ssh msfadmin@192.168.1.50
# Password: msfadmin
```
✅ Login successful — default credentials confirmed working.

---

## Key Takeaways

- Full root access available immediately — no privilege escalation needed
- Kernel is 15+ years out of date — massive attack surface
- Default credentials (`msfadmin:msfadmin`) work on SSH
- `/etc/shadow` readable with weak MD5 hashes — all accounts crackable
