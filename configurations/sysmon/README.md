# Sysmon Configuration

## SwiftOnSecurity Config

This lab uses the **SwiftOnSecurity Sysmon configuration**, the industry-standard tuned config for security visibility.

### Why This Config?

The default Sysmon config logs almost nothing useful. SwiftOnSecurity's config:
- Filters out noisy events (system processes, signed binaries)
- Captures all PowerShell execution
- Logs network connections from suspicious processes
- Monitors registry persistence locations
- Tracks DLL injection indicators
- Maps cleanly to MITRE ATT&CK techniques

### Download

The config file is too long to redistribute here. Get the latest version from the official source:

**URL:** https://github.com/SwiftOnSecurity/sysmon-config

**Direct download:**
```
https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml
```

### Installation

```powershell
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

### Verify

```powershell
Get-Service Sysmon64
# Should show: Status: Running

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

### Update Existing Sysmon Config

If Sysmon is already installed and you want to update the config:

```powershell
.\Sysmon64.exe -c sysmonconfig-export.xml
```

---

## Key Sysmon Event IDs

| Event ID | Description | MITRE Relevance |
|---|---|---|
| **1** | Process creation | Almost every technique |
| **3** | Network connection | C2, lateral movement |
| **7** | Image (DLL) loaded | DLL injection |
| **10** | Process accessed | Credential dumping (LSASS) |
| **11** | File created | Persistence, staging |
| **13** | Registry value set | Persistence |
| **22** | DNS query | C2 callbacks |

---

## Alternative Configs

If SwiftOnSecurity doesn't fit your environment:

- **Olaf Hartong's Modular Config** — https://github.com/olafhartong/sysmon-modular
- **Florian Roth's Config** — https://github.com/Neo23x0/sysmon-config
