# VM Specifications

Hardware allocations and configuration details for each virtual machine in the lab.

---

## 🖥️ DC01 — Domain Controller

| Setting | Value |
|---|---|
| **OS** | Windows Server 2022 Standard Evaluation (Server Core) |
| **IP Address** | 192.168.10.20 |
| **Hostname** | DC01 |
| **Domain** | soc.local |
| **NetBIOS Name** | SOC |
| **RAM** | 2–4 GB |
| **CPU** | 2 cores |
| **Disk** | 50 GB |
| **Network Adapter** | Custom: VMnet10 |
| **Roles** | AD DS, DNS Server |

---

## 🐧 SPLUNK-SERVER — SIEM

| Setting | Value |
|---|---|
| **OS** | Ubuntu 22.04 LTS Server |
| **IP Address** | 192.168.10.10 |
| **Hostname** | splunkserver |
| **RAM** | 4–8 GB |
| **CPU** | 2–4 cores |
| **Disk** | 60 GB (minimum recommended) |
| **Network Adapter** | Custom: VMnet10 |
| **Software** | Splunk Enterprise 10.x |
| **Ports** | 8000 (Web UI), 9997 (Forwarder), 8089 (Mgmt) |

---

## 💻 WIN-TARGET — Victim Endpoint

| Setting | Value |
|---|---|
| **OS** | Windows 11 Pro |
| **IP Address** | 192.168.10.30 |
| **Hostname** | WIN-TARGET (or DESKTOP-* default) |
| **Domain** | soc.local (joined) |
| **RAM** | 2–4 GB |
| **CPU** | 2 cores |
| **Disk** | 50 GB |
| **Network Adapter** | Custom: VMnet10 |
| **Software** | Sysmon, Splunk Universal Forwarder, Atomic Red Team |

---

## ⚔️ KALI — Attacker (Upcoming)

| Setting | Value |
|---|---|
| **OS** | Kali Linux Rolling 2026 |
| **IP Address** | 192.168.10.50 |
| **RAM** | 2–4 GB |
| **CPU** | 2 cores |
| **Disk** | 30–40 GB |
| **Network Adapter** | Custom: VMnet10 (isolated only) |
| **Tools** | Hydra, Nmap, Metasploit, CrackMapExec |

---

## 🌐 Network Configuration

| Setting | Value |
|---|---|
| **VMware Network** | VMnet10 |
| **Network Type** | Host-only |
| **Subnet** | 192.168.10.0/24 |
| **Subnet Mask** | 255.255.255.0 |
| **Gateway** | 192.168.10.1 (host adapter) |
| **DHCP** | Disabled (manual static IPs) |

---

## 💾 Host PC Requirements

| Component | Minimum | Recommended |
|---|---|---|
| **RAM** | 16 GB | 32 GB |
| **CPU** | Quad-core w/ VT-x | 8-core |
| **Storage** | 250 GB SSD | 500 GB SSD |
| **Virtualization** | Enabled in BIOS | Enabled in BIOS |
