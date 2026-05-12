# Atomic Red Team Tests Executed

Log of MITRE ATT&CK simulations run against the Windows target, with their detection status.

---

## 🧪 Tests Run

### ✅ T1082 — System Information Discovery
**Tactic:** Discovery
**Command:** `Invoke-AtomicTest T1082 -TestNumbers 1`
**Result:** Successful execution. `systeminfo.exe` ran, output captured.
**Sysmon Event:** EventID=1, Image=`systeminfo.exe`
**Detection SPL:**
```spl
index=endpoint sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventID>\d+)</EventID>"
| search EventID=1
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| search Image="*systeminfo.exe"
```

---

### ❌ T1059.001 Test 1 — Mimikatz via PowerShell
**Tactic:** Execution / Credential Access
**Command:** `Invoke-AtomicTest T1059.001 -TestNumbers 1`
**Result:** **Blocked by Windows Defender** — "Access is denied"
**Lesson:** Defender catches known offensive tools. Either disable Defender for the test, or use cleaner techniques.

---

### ❌ T1059.001 Test 3 — Bloodhound from Memory
**Tactic:** Execution / Discovery
**Command:** `Invoke-AtomicTest T1059.001 -TestNumbers 3`
**Result:** **Blocked by Windows Defender**
**Lesson:** Same as above — Defender signatures block these.

---

## 📋 Recommended Test Plan

The following Atomic tests are **safe** to run without disabling Defender:

| Technique | Name | Detection Difficulty |
|---|---|---|
| T1082 | System Information Discovery | Easy |
| T1033 | System Owner/User Discovery | Easy |
| T1007 | System Service Discovery | Easy |
| T1016 | System Network Configuration Discovery | Easy |
| T1018 | Remote System Discovery | Easy |
| T1087 | Account Discovery | Medium |
| T1057 | Process Discovery | Medium |

To run them:

```powershell
# Each test below is independent
Invoke-AtomicTest T1082 -TestNumbers 1
Invoke-AtomicTest T1033 -TestNumbers 1
Invoke-AtomicTest T1007 -TestNumbers 1
Invoke-AtomicTest T1016 -TestNumbers 1
Invoke-AtomicTest T1018 -TestNumbers 1
Invoke-AtomicTest T1087 -TestNumbers 1
Invoke-AtomicTest T1057 -TestNumbers 1
```

For higher-impact tests (credential dumping, exploitation), temporarily disable real-time protection:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
# Run riskier tests
Invoke-AtomicTest T1003.001 -TestNumbers 1
Set-MpPreference -DisableRealtimeMonitoring $false
```

---

## 🧹 Cleanup

Always clean up after each test to keep the system in a known state:

```powershell
Invoke-AtomicTest <TechniqueID> -TestNumbers <N> -Cleanup
```

---

## 📚 Resources

- **Atomic Red Team Repo:** https://github.com/redcanaryco/atomic-red-team
- **Technique Index:** Look in `C:\AtomicRedTeam\atomics\` for all 600+ technique folders
- **MITRE ATT&CK Reference:** https://attack.mitre.org
