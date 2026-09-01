# 01 — Lab Setup & Network Configuration

## Objective
Set up a safe, isolated home lab with Kali Linux (attacker) and Metasploitable2 (victim) able to communicate on the same network.

---

## Step 1 — Update Kali Linux
```bash
sudo apt update && sudo apt upgrade -y
```
- `apt update` — refreshes package list (1–2 mins)
- `apt upgrade -y` — installs all updates (10–30 mins on fresh install)

---

## Step 2 — Install Metasploitable2
- Download: https://sourceforge.net/projects/metasploitable/files/Metasploitable2/
- Import into VirtualBox (File → Import Appliance)
- Default login: `msfadmin / msfadmin`

---

## Step 3 — Network Configuration

**Problem:** Default VirtualBox NAT mode gives both VMs `10.0.2.15` — they cannot communicate with each other on plain NAT.

**Solution:** Switch both VMs to **Bridged Adapter**.

### Kali Settings (Settings → Network → Adapter 1):
| Setting | Value |
|---------|-------|
| Attached to | Bridged Adapter |
| Name | Intel(R) Centrino(R) Advanced-N 6205 |
| Promiscuous Mode | Allow All |

### Metasploitable2 Settings:
| Setting | Value |
|---------|-------|
| Attached to | Bridged Adapter |
| Name | Intel(R) Centrino(R) Advanced-N 6205 |
| Promiscuous Mode | Allow All |

---

## Step 4 — Verify IPs

**Kali:**
```bash
ifconfig
# eth0: 192.168.1.80
```

**Metasploitable2:**
```bash
ifconfig
# eth0: 192.168.1.50
```

---

## Step 5 — Test Connectivity
```bash
ping 192.168.1.50
# 64 bytes from 192.168.1.50: icmp_seq=1 ttl=64 time=0.xxx ms
```
✅ Both machines connected and ready.

---

## Network Diagram
```
[Kali Linux 192.168.1.80]  ←── LAN ──→  [Metasploitable2 192.168.1.50]
         (Attacker)                              (Target)
               ↑
         [Home Router]
         192.168.1.1
               ↑
          [Internet]
```

---

## ⚠️ Safety Note
Metasploitable2 is **intentionally vulnerable**. Never expose it to the internet or a public network.
