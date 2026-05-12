# Attack → Detection Mapping

A reference table mapping every Atomic Red Team test in the lab to the SPL query that detects it.

---

## 🎯 Complete Mapping

| MITRE ID | Technique | Tactic | Atomic Test | Detection Query | Status |
|---|---|---|---|---|---|
| T1082 | System Info Discovery | Discovery | T1082-1 | `attack-detections.spl #1` | ✅ Tested |
| T1059.001 | Encoded PowerShell | Execution | Manual + T1059.001-* | `attack-detections.spl #2` | 🔄 Pending |
| T1110 | Brute Force | Credential Access | Via Kali (Hydra) | `attack-detections.spl #3` | ⏳ Next |
| T1136.001 | Local Admin Creation | Persistence | T1136.001-1 | `attack-detections.spl #4` | ⏳ Pending |
| T1003.001 | LSASS Memory Dump | Credential Access | T1003.001-1 | `attack-detections.spl #5` | ⏳ Pending |
| T1053.005 | Scheduled Task | Persistence | T1053.005-1 | `attack-detections.spl #6` | ⏳ Pending |
| T1566.001 | Phishing Attachment | Initial Access | Manual simulation | `attack-detections.spl #7` | ⏳ Pending |
| T1105 | Ingress Tool Transfer | Command & Control | T1105-* | `attack-detections.spl #8` | ⏳ Pending |
| Various | LOLBins Abuse | Defense Evasion | Multiple | `attack-detections.spl #10` | ⏳ Pending |

Legend: ✅ Tested & Detected | 🔄 In Progress | ⏳ Future Phase

---

## 🔥 The Kill Chain — Combining Detections

A real attack uses multiple techniques chained together. Practice detecting the full chain:

```
1. Initial Access
   └─ T1566.001 (Phishing Attachment)
      Detection: Office app spawning powershell.exe

2. Execution
   └─ T1059.001 (PowerShell)
      Detection: Encoded command or download cradle

3. Discovery
   └─ T1082 (System Info Discovery)
      Detection: systeminfo.exe execution

4. Credential Access
   └─ T1003.001 (LSASS Memory Dump)
      Detection: Process access to lsass.exe

5. Persistence
   └─ T1053.005 (Scheduled Task)
      Detection: schtasks.exe + Event ID 4698

6. Lateral Movement (future)
   └─ T1021.001 (RDP)
      Detection: Event ID 4624 Logon Type 10

7. Exfiltration (future)
   └─ T1041 (Exfil over C2)
      Detection: Sysmon EID 3 to suspicious IP
```

---

## 📊 Building a MITRE Coverage Heatmap

Once you have most detections in place, build a dashboard panel that visualizes coverage:

```spl
| rest /servicesNS/-/-/saved/searches 
| search action.notable.param.rule_name="*"
| rex field=description "T(?<technique_id>\d+(\.\d+)?)"
| stats count by technique_id
```

Then plot on a MITRE Navigator overlay (https://mitre-attack.github.io/attack-navigator/).
