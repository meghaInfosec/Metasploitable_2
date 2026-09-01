# 05 — SMB Enumeration

## What is SMB?
SMB (Server Message Block) is a **LAN file sharing protocol** — no internet needed. Linux uses **Samba** as its SMB implementation.

---

## Tool 1 — smbclient (Share Listing)

```bash
smbclient -L //192.168.1.50
# Password: [press Enter]
```

Output:
```
Anonymous login successful

Sharename    Type    Comment
---------    ----    -------
print$       Disk    Printer Drivers
tmp          Disk    oh noes!
opt          Disk
IPC$         IPC     IPC Service (Samba 3.0.20-Debian)
ADMIN$       IPC     IPC Service (Samba 3.0.20-Debian)

Workgroup    Master
---------    -------
WORKGROUP    METASPLOITABLE
```

---

## Tool 2 — Accessing tmp Share

```bash
smbclient //192.168.1.50/tmp
# Password: [press Enter]

smb: \> ls
smb: \> put test.txt      # Upload file
smb: \> get <filename>    # Download file
```

✅ **Read and write access confirmed — no credentials needed!**

---

## Tool 3 — enum4linux (Deep Enumeration)

```bash
enum4linux 192.168.1.50
```

### Users Dumped (34 total):
```
root, msfadmin, user, www-data, mysql, postgres,
tomcat55, ftp, sshd, telnetd, proftpd, distccd,
games, nobody, bind, proxy, syslog, daemon, bin,
mail, news, man, lp, gnats, backup, libuuid,
postfix, klog, service, list, irc, sync, uucp, sys
```

### OS Information:
```
Samba 3.0.20-Debian
OS version: 4.9
```

### Password Policy:
```
Minimum password length: 5
Password Complexity: Disabled
Account Lockout Threshold: None
Maximum password age: Not Set
```

---

## Key Findings

| Finding | Risk |
|---------|------|
| Anonymous SMB login | 🔴 Critical |
| 34 usernames exposed | 🔴 Critical |
| No account lockout | 🔴 Critical |
| Password complexity disabled | 🔴 Critical |
| Samba 3.0.20 version exposed | 🔴 Critical |
| tmp share read/write access | 🟠 High |

---

## Remediation
1. Disable anonymous SMB: `restrict anonymous = 2`
2. Enable account lockout (max 5 attempts)
3. Enforce password complexity — min 12 chars
4. Upgrade Samba to latest version
5. Restrict SMB port 445 to trusted IPs
