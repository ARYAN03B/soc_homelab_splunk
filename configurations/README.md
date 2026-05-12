# Configuration Files

Production-tested configuration files from the actual lab build.

## 📁 Structure

```
03-configurations/
├── splunk-forwarder/    → Windows endpoint configs
│   ├── inputs.conf      → What logs to collect
│   ├── outputs.conf     → Where to send logs
│   └── server.conf      → Forwarder identity
├── sysmon/
│   └── README.md        → Sysmon config source & install
└── splunk-server/
    └── server.conf      → Lab-specific Splunk tweaks
```

## Quick Reference

### Forwarder Configs (Windows Target)

Place in: `C:\Program Files\SplunkUniversalForwarder\etc\system\local\`

| File | Purpose |
|---|---|
| `inputs.conf` | Defines which Event Log channels to monitor |
| `outputs.conf` | Points the forwarder at the Splunk server |
| `server.conf` | Overrides cached hostname |

**After any config change:** `Restart-Service SplunkForwarder`

### Splunk Server Config (Ubuntu)

Place in: `/opt/splunk/etc/system/local/`

| File | Purpose |
|---|---|
| `server.conf` | Lowers disk space threshold for lab |

**After any config change:** `sudo /opt/splunk/bin/splunk restart`

---
