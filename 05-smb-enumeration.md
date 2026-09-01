# 05 — SMB Enumeration

## 🎯 Goal of This Step
Check whether the Victim Machine's file-sharing service (SMB) leaks information or allows access without a login.

## 🔑 Machines in This Step

| Role | Machine | IP Address |
|------|---------|------------|
| 🟥 Attacker | Kali Linux | `192.168.1.80` |
| 🟩 Victim / Target | Metasploitable 2 | `192.168.1.50` |

---

## What is SMB?
SMB (Server Message Block) is a **LAN file sharing protocol** — no internet needed, just a local network, like sharing folders between office computers. Linux uses **Samba** as its version of SMB. Here it runs on the Victim Machine.

---

## Tool 1 — smbclient (Share Listing) — run FROM 🟥 Attacker, pointed AT 🟩 Victim

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

**In plain English:** This command asks the Victim Machine "what shared folders do you have?" — and it answers without ever asking for a real password.

---

## Tool 2 — Accessing the tmp Share (🟥 Attacker connects to a folder on 🟩 Victim)

```bash
smbclient //192.168.1.50/tmp
# Password: [press Enter]

smb: \> ls
smb: \> put test.txt      # Upload a file to the Victim Machine
smb: \> get <filename>    # Download a file from the Victim Machine
```

✅ **Read and write access confirmed — no credentials needed!**

**In plain English:** The Attacker can freely drop files onto the Victim Machine's shared folder, or pull files off it — like a public locker with no lock.

---

## Tool 3 — enum4linux (Deep Enumeration) — run FROM 🟥 Attacker, pointed AT 🟩 Victim

```bash
enum4linux 192.168.1.50
```

### Users Dumped From the Victim Machine (34 total):
```
root, msfadmin, user, www-data, mysql, postgres,
tomcat55, ftp, sshd, telnetd, proftpd, distccd,
games, nobody, bind, proxy, syslog, daemon, bin,
mail, news, man, lp, gnats, backup, libuuid,
postfix, klog, service, list, irc, sync, uucp, sys
```

**In plain English:** This tool asks the Victim Machine to list every user account it has — information that should normally be private, since it hands an attacker a ready-made list of usernames to try passwords against.

### OS Information (of the Victim Machine):
```
Samba 3.0.20-Debian
OS version: 4.9
```

### Password Policy (on the Victim Machine):
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
| Anonymous SMB login on Victim Machine | 🔴 Critical |
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
