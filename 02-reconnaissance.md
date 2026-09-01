# 02 — Reconnaissance & System Enumeration

## 🎯 Goal of This Step
Scan the Victim Machine from the outside first (to find open doors/services), then — after access was gained using the exploit covered in [04-vsftpd-backdoor.md](./04-vsftpd-backdoor.md) — look around inside it to see exactly what an attacker could find.

> 📌 **Note on order:** The deep system enumeration below (reading `/etc/shadow`, checking `whoami`, etc.) happens *after* root access was obtained via the vsftpd backdoor exploit in file 04. It's placed here early in the repo because "recon" is normally the first phase of a pentest — but the actual root shell used below was achieved later in the walkthrough. If you're following along command-by-command, read [03](./03-ftp-exploitation.md) and [04](./04-vsftpd-backdoor.md) first.

## 🔑 Machines in This Step

| Role | Machine | IP Address |
|------|---------|------------|
| 🟥 Attacker | Kali Linux | `192.168.1.80` |
| 🟩 Victim / Target | Metasploitable 2 | `192.168.1.50` |

---

## Part A — Scanning From Outside (🟥 Attacker Machine → 🟩 Victim Machine)

```bash
nmap -sV 192.168.1.50
```
**In plain English:** Nmap checks every "door" (port) on the Victim Machine to see which ones are open and what software is running behind them.

Key open ports discovered on the **Victim Machine**:

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

## Part B — Once Inside (🟥 Attacker Machine, operating through a shell it gained ON the 🟩 Victim Machine)

After root access was achieved (see [04-vsftpd-backdoor.md](./04-vsftpd-backdoor.md)), these commands were run — they execute *on the Victim Machine*, controlled remotely by the Attacker Machine:

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
✅ Session was already running with **root** privileges on the Victim Machine — no privilege escalation was required.

### System Fingerprinting (commands run ON the Victim Machine)
```bash
whoami       # root
id           # uid=0(root) gid=0(root)
uname -a     # Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
hostname     # metasploitable
```

| Finding | Value |
|---------|-------|
| OS | Ubuntu-based Metasploitable build |
| Kernel | 2.6.24-16-server (2008) — multiple known local privilege escalation bugs |
| Architecture | 32-bit (i686) |
| User context | root (uid=0) |

**In plain English:** These commands ask the Victim Machine "who am I logged in as, and what system are you?" The answers confirm the Attacker has the highest level of access (`root`) and that the Victim Machine is running a 15+ year old, unpatched operating system.

---

## Network Information (on 🟩 Victim Machine)
```bash
ifconfig
```

| Interface | IP Address | Notes |
|-----------|-----------|-------|
| `eth0` | 192.168.1.50 | Primary interface |
| `lo` | 127.0.0.1 | Loopback |

---

## Credential Harvesting — /etc/shadow (read ON the 🟩 Victim Machine, by the 🟥 Attacker)

```bash
cat /etc/shadow
```
**In plain English:** This is the file where Linux stores every user's password — not in plain text, but scrambled ("hashed"). Normally only an admin can read this file. Because the Attacker already has root, it can read it freely.

Accounts found with active password hashes (**MD5-crypt `$1$` format** — weak and crackable with `john` or `hashcat`):
- `root`, `sys`, `klog`, `msfadmin`, `postgres`, `user`, `service`

> **Note:** MD5-crypt hashes are considered weak by modern standards. SHA-512 or bcrypt should be used instead.

---

## SSH Access with Default Credentials (🟥 Attacker Machine → 🟩 Victim Machine)
```bash
ssh msfadmin@192.168.1.50
# Password: msfadmin
```
✅ Login successful — default credentials confirmed working.

**In plain English:** The Victim Machine still uses its factory-default username and password, like never changing the default WiFi router password — anyone who knows the defaults can walk right in.

---

## Key Takeaways

- Full root access available on the Victim Machine — no privilege escalation needed
- Victim's kernel is 15+ years out of date — massive attack surface
- Default credentials (`msfadmin:msfadmin`) work on SSH
- `/etc/shadow` readable with weak MD5 hashes — all accounts on the Victim Machine are crackable
