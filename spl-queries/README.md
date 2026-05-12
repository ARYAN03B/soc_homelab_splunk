# SPL Queries

All Splunk Search Processing Language queries used in the lab.

## 📁 Files

| File | Purpose | When to use |
|---|---|---|
| `data-validation.spl` | Verify data is flowing into Splunk | During initial setup |
| `sysmon-field-extraction.spl` | Parse Sysmon XML into fields | Whenever working with Sysmon data |
| `attack-detections.spl` | MITRE-mapped detection rules | Save as Splunk Alerts |
| `threat-hunting.spl` | Proactive hunting queries | Manual investigation |

## 🎯 Detection Coverage (MITRE ATT&CK)

| Technique | Name | File | Detection # |
|---|---|---|---|
| T1082 | System Information Discovery | attack-detections | 1 |
| T1059.001 | PowerShell — Encoded Command | attack-detections | 2, 8 |
| T1110 | Brute Force | attack-detections | 3, 9 |
| T1136.001 | Local Account Creation | attack-detections | 4 |
| T1003.001 | LSASS Memory Access | attack-detections | 5 |
| T1053.005 | Scheduled Task | attack-detections | 6 |
| T1566.001 | Spear Phishing Attachment | attack-detections | 7 |
| T1105 | Ingress Tool Transfer | attack-detections | 8 |
| Various | LOLBins | attack-detections | 10 |
| Various | Registry Persistence | threat-hunting | 7 |
| T1071.004 | DNS C2 | threat-hunting | 8 |
| T1543.003 | New Service | threat-hunting | 9 |
| T1087 | Account Discovery | threat-hunting | 10 |

## 💡 How to Use

1. Open Splunk Web UI → Search & Reporting
2. Copy a query from any `.spl` file
3. Paste it into the search bar
4. Set time range (typically "Last 15 minutes" or "Last hour")
5. Click Search
6. Click the **Statistics** tab to see results

## 🚨 Converting to Alerts

To turn a detection into a fired alert:

1. Run the query in Search & Reporting
2. Click **Save As → Alert**
3. Configure:
   - **Alert type:** Scheduled
   - **Run every:** 5 minutes
   - **Trigger condition:** Number of Results > 0
   - **Action:** Add to Triggered Alerts (or Email)
4. Save

## 📝 Note on Field Extraction

These queries use `rex` (regex) to extract Sysmon XML fields. If you install the **Splunk Add-on for Microsoft Sysmon**, fields auto-extract and you can drop the `rex` lines for cleaner queries. The regex versions still work either way as a fallback.
