# 04 — SMB Enumeration

## What is SMB?

SMB (Server Message Block) is a **network file sharing protocol** that allows computers on the same LAN to share files, folders, and printers — **without internet**.

**Analogy:** SMB is like roads inside a neighbourhood — it only needs local connectivity, not the highway (internet).

- Windows uses SMB natively
- Linux uses **Samba** — the open-source implementation of SMB

---

## What is SMB Enumeration?

Enumeration = **gathering intelligence** about the target before attacking.

SMB Enumeration reveals:
- Shared folders (shares)
- Usernames and groups
- OS version and Samba version
- Password policy weaknesses
- Machine name and workgroup

**Analogy:** Like a private detective who observes a building — noting entrances, who works there, what the security policy is — all without breaking in.

---

## Tool 1 — smbclient (Share Listing)

### Command:
```bash
smbclient -L //192.168.1.50
```

- `-L` = List all available shares
- When asked for password → just press Enter

### Output:
```
Anonymous login successful

Sharename    Type    Comment
---------    ----    -------
print$       Disk    Printer Drivers
tmp          Disk    oh noes!
opt          Disk
IPC$         IPC     IPC Service (metasploitable server (Samba 3.0.20-Debian))
ADMIN$       IPC     IPC Service (metasploitable server (Samba 3.0.20-Debian))

Workgroup    Master
---------    -------
WORKGROUP    METASPLOITABLE
```

### Key Findings from smbclient:

| Share | Risk | Reason |
|-------|------|--------|
| `tmp` | 🔴 High | Comment "oh noes!" — open read/write |
| `ADMIN$` | 🔴 High | Administrative share visible |
| `IPC$` | 🟡 Medium | Remote management interface exposed |
| `print$` | 🟢 Low | Printer drivers only |
| Samba 3.0.20 | 🔴 Critical | Old, vulnerable version exposed |

---

## Tool 2 — Accessing the tmp Share

### Command:
```bash
smbclient //192.168.1.50/tmp
```

When asked for password → press Enter

### Inside the share:
```bash
smb: \> ls       # List files
smb: \> pwd      # Current directory
smb: \> put test.txt    # Upload a file
smb: \> get <filename>  # Download a file
```

### Files found in tmp:
```
.ICE-unix      DH    Hidden directory
.X11-unix      DH    Hidden directory
.X0-lock       HR    11 bytes
4567.jsvc_up   R     Apache Tomcat running
```

### File Upload Test:
```bash
# On Kali first:
echo "hacked by megha" > /root/test.txt

# Then in SMB session:
smb: \> put test.txt
putting file test.txt as \test.txt (1.2 kB/s)
```

✅ **Write access confirmed — file successfully uploaded to target!**

---

## Tool 3 — enum4linux (Deep Enumeration)

### What is enum4linux?
A tool that **automatically extracts** everything available from an SMB/Samba target — users, groups, shares, password policy, OS details — without valid credentials.

### Command:
```bash
enum4linux 192.168.1.50
```

### Results:

#### Users Found (34 total):
```
root, msfadmin, user, www-data, mysql, postgres,
tomcat55, ftp, sshd, telnetd, proftpd, distccd,
games, nobody, bind, proxy, syslog, daemon, bin,
mail, news, man, lp, gnats, backup, libuuid,
postfix, klog, service, list, irc, sync, uucp, sys
```

#### OS Information:
```
Samba 3.0.20-Debian
OS version: 4.9
Platform ID: 500
```

#### Password Policy:
```
Minimum password length: 5
Password Complexity: Disabled
Account Lockout Threshold: None
Maximum password age: Not Set
```

#### Share Mapping:
```
//192.168.1.50/tmp    Mapping: OK  Listing: OK  ← Fully accessible
//192.168.1.50/print$ Mapping: DENIED           ← Restricted
//192.168.1.50/ADMIN$ Mapping: DENIED           ← Restricted
```

---

## Critical Findings from enum4linux

| Finding | Risk | Impact |
|---------|------|--------|
| 34 usernames exposed | 🔴 Critical | Ready-made list for brute force |
| root user confirmed | 🔴 Critical | Highest privilege account identified |
| No account lockout | 🔴 Critical | Unlimited password guessing allowed |
| Password complexity disabled | 🔴 Critical | Weak passwords likely |
| Min password length: 5 | 🔴 Critical | Very short passwords allowed |
| Samba 3.0.20 version exposed | 🔴 Critical | Known exploitable version |
| tmp share fully accessible | 🟠 High | Read/write without credentials |

---

## Remediation

1. **Disable anonymous SMB access** in smb.conf:
   ```
   restrict anonymous = 2
   ```
2. **Hide share names** from unauthenticated users
3. **Enforce strong password policy** — minimum 12 chars, complexity required
4. **Enable account lockout** — max 5 attempts, 30-minute lockout
5. **Upgrade Samba** to latest patched version
6. **Restrict SMB to specific IPs** using firewall rules
7. **Disable SMB on port 445** if not needed
