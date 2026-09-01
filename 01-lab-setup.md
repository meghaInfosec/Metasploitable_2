# 01 — Lab Setup

## Objective
Set up a safe, isolated home lab with Kali Linux (attacker) and Metasploitable2 (victim) able to communicate with each other.

---

## Tools Required
- **VirtualBox** — hypervisor to run both VMs
- **Kali Linux 2026.1** — attacker machine
- **Metasploitable2** — intentionally vulnerable target machine

---

## Step 1 — Update Kali Linux

Before starting, always update Kali to get the latest tools and patches:

```bash
sudo apt update && sudo apt upgrade -y
```

- `apt update` — refreshes the package list (fast, 1–2 mins)
- `apt upgrade -y` — installs all updates (may take 10–30 mins on fresh install)

---

## Step 2 — Install Metasploitable2

- Download from: https://sourceforge.net/projects/metasploitable/files/Metasploitable2/
- Import the VM into VirtualBox (File → Import Appliance)
- Default credentials: `msfadmin / msfadmin`

---

## Step 3 — Network Configuration

Both VMs must be on the **same network** to communicate.

**Problem:** By default, VirtualBox uses **NAT** mode for VMs. Each VM gets `10.0.2.15` — but they cannot talk to each other on NAT.

**Solution:** Switch both VMs to **Bridged Adapter** mode.

### Kali Linux Network Settings:
| Setting | Value |
|---------|-------|
| Attached to | Bridged Adapter |
| Name | Intel(R) Centrino(R) Advanced-N 6205 |
| Promiscuous Mode | Allow All |

### Metasploitable2 Network Settings:
| Setting | Value |
|---------|-------|
| Attached to | Bridged Adapter |
| Name | Intel(R) Centrino(R) Advanced-N 6205 |
| Promiscuous Mode | Allow All |

---

## Step 4 — Verify IP Addresses

**On Kali:**
```bash
ifconfig
```
Result: `eth0: 192.168.1.80`

**On Metasploitable2:**
```bash
ifconfig
```
Result: `eth0: 192.168.1.50`

---

## Step 5 — Test Connectivity

From Kali terminal:
```bash
ping 192.168.1.50
```

Expected output:
```
64 bytes from 192.168.1.50: icmp_seq=1 ttl=64 time=0.xxx ms
```

✅ Both machines are now connected and ready for penetration testing.

---

## Network Diagram

```
[Kali Linux]                    [Metasploitable2]
192.168.1.80    ←── LAN ──→    192.168.1.50
  (Attacker)                      (Target)
       ↑
  [Home Router]
  192.168.1.1
       ↑
  [Internet]
```

---

## ⚠️ Safety Note
Metasploitable2 is **intentionally vulnerable**. Keep it on a private/isolated network. Never expose it to the internet or public Wi-Fi.
