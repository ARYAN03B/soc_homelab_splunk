# 🛡️ SOC Homelab with Splunk SIEM

A production-grade Security Operations Center homelab built from scratch using Splunk Enterprise as the SIEM, simulating real enterprise threat detection workflows mapped to the MITRE ATT&CK framework.

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Splunk](https://img.shields.io/badge/Splunk-10.x-orange)
![Sysmon](https://img.shields.io/badge/Sysmon-15.x-blue)
![MITRE](https://img.shields.io/badge/MITRE-ATT%26CK-red)

---

## 🎯 Project Goals

- Build a complete SOC analyst training environment from scratch
- Generate realistic attack telemetry using Atomic Red Team
- Detect attacks in real-time with Splunk SPL queries
- Map detections to MITRE ATT&CK techniques
- Practice incident response workflows

---

## 🏗️ Architecture

| VM | IP Address | Role | OS |
|---|---|---|---|
| **DC01** | 192.168.10.20 | Domain Controller | Windows Server 2022 |
| **SPLUNK-SERVER** | 192.168.10.10 | SIEM | Ubuntu 22.04 LTS |
| **WIN-TARGET** | 192.168.10.30 | Victim Endpoint | Windows 11 Pro |
| **KALI** | 192.168.10.50 | Attacker (upcoming) | Kali Linux |

All VMs run on VMware Workstation, isolated on **VMnet10** (192.168.10.0/24).

---

## 🛠️ Tools & Technologies

| Category | Tool |
|---|---|
| **SIEM** | Splunk Enterprise 10.x |
| **Endpoint Telemetry** | Sysmon + SwiftOnSecurity config |
| **Log Forwarding** | Splunk Universal Forwarder |
| **Attack Simulation** | Atomic Red Team (Red Canary) |
| **Identity** | Active Directory Domain Services |
| **Detection Framework** | MITRE ATT&CK |
| **Virtualization** | VMware Workstation Pro |

---

## 📋 Data Flow

```
WIN-TARGET (192.168.10.30)
    ↓ Sysmon captures process creation, network, file events
    ↓ Splunk Universal Forwarder reads Event Log channels
    ↓ Sends via TCP port 9997
    ↓
SPLUNK-SERVER (192.168.10.10)
    ↓ Parses XML, indexes events in "endpoint" index
    ↓ SPL queries detect attacks
    ↓ Alerts fire on suspicious behavior
```

---

## 📂 Repository Structure

```
soc-homelab-splunk/
├── 01-architecture/         → Network diagrams and VM specs
├── 02-setup-guides/         → Step-by-step build instructions
├── 03-configurations/       → All working config files
├── 04-spl-queries/          → Detection and hunting queries
├── 05-atomic-tests/         → MITRE ATT&CK simulations
├── 06-dashboards/           → Splunk dashboard XMLs
├── 07-screenshots/          → Visual proof of detections
└── 08-future-roadmap.md     → What's next
```

---

## 🚧 Current Status

- ✅ **Phase 1:** Infrastructure complete (DC, Splunk, Windows target)
- ✅ **Phase 2:** Sysmon + Forwarder pipeline operational
- ✅ **Phase 3:** Atomic Red Team integrated
- 🔄 **Phase 4:** Kali attacker VM (in progress)
- ⏳ **Phase 5:** Detection rules + alerts
- ⏳ **Phase 6:** SOC dashboards
- ⏳ **Phase 7:** Ubuntu Linux target
- ⏳ **Phase 8:** Sigma rule conversion

---

## 🚀 Quick Start

1. Read [`02-setup-guides/01-vmware-network-setup.md`](02-setup-guides/01-vmware-network-setup.md)
2. Follow each numbered guide in order
3. Hit a problem? Check [`02-setup-guides/05-troubleshooting.md`](02-setup-guides/05-troubleshooting.md) — every real issue I encountered is documented

---

## 📚 References & Credits

- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) — Red Canary
- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
- [SigmaHQ](https://github.com/SigmaHQ/sigma) — Generic detection rules
- [MITRE ATT&CK](https://attack.mitre.org) — Adversary tactics & techniques
- [Splunk Security Content](https://research.splunk.com)

---

## 📜 License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.

---

## 🤝 Contributing

This is a personal learning project, but suggestions, corrections, and ideas are welcome via Issues or Pull Requests.

---

**Built by a SOC analyst in training, documented for everyone else trying to do the same.**
