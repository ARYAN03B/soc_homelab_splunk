# Architecture Overview

This folder contains the high-level design of the SOC homelab.

## Files

- **vm-specifications.md** — Hardware and OS specs for every VM
- **network-diagram.png** — Visual network topology (add your own)
- **data-flow-diagram.png** — How logs flow through the system (add your own)

## Lab Topology

```
┌───────────────────────────────────────────────────────────────┐
│                    VMnet10 - 192.168.10.0/24                  │
│                                                               │
│  ┌──────────────┐   ┌────────────────┐   ┌────────────────┐   │
│  │ DC01         │   │ SPLUNK-SERVER  │   │ WIN-TARGET     │   │
│  │ 192.168.10.20│   │ 192.168.10.10  │   │ 192.168.10.30  │   │
│  │ Win Srv 2022 │   │ Ubuntu 22.04   │   │ Windows 11 Pro │   │
│  │ AD/DNS       │   │ Splunk SIEM    │   │ Sysmon+Forwdr  │   │
│  └──────┬───────┘   └────────▲───────┘   └────────┬───────┘   │
│         │                    │                    │           │
│         │ Auth               │ Logs (9997)        │           │
│         └────────────────────┴────────────────────┘           │
│                              │                                │
│                    ┌─────────▼─────────┐                      │
│                    │      KALI         │                      │
│                    │ 192.168.10.50     │                      │
│                    │ Attacker          │                      │
│                    └───────────────────┘                      │
└───────────────────────────────────────────────────────────────┘
```

## Data Flow

1. Attack runs on **WIN-TARGET**
2. **Sysmon** captures process creation, network, file events
3. **Splunk Universal Forwarder** reads Windows Event Log channels
4. Events sent to **Splunk Server** on TCP/9997
5. **Splunk** parses, indexes, and stores in `endpoint` index
6. Analyst runs **SPL queries** to find attack indicators
7. **Alerts** fire on suspicious patterns
