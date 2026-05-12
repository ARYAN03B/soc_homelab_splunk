# Setup Guides

Follow these guides **in order** to build the SOC homelab from scratch.

| # | Guide | Time |
|---|---|---|
| 01 | [VMware Network Setup](01-vmware-network-setup.md) | 15 min |
| 02 | [Domain Controller Setup](02-domain-controller-setup.md) | 45 min |
| 03 | [Splunk Server Setup](03-splunk-server-setup.md) | 60 min |
| 04 | [Windows Target Setup](04-windows-target-setup.md) | 60 min |
| 05 | [Troubleshooting Guide](05-troubleshooting.md) | Reference |

---

## Recommended VM Power-On Order

```
1st → DC01 (Domain Controller)
        ↓ wait 1–2 min for AD services
2nd → SPLUNK-SERVER (Ubuntu)
        ↓ wait 30 sec for Splunk listener
3rd → WIN-TARGET (Windows 11)
        ↓ ready for use
4th → KALI (when built)
```

The DC must come up first because the Windows Target depends on it for DNS and authentication. Splunk needs to listen on port 9997 before the Forwarder tries to connect.
