# Atomic Red Team Integration

The lab uses [Red Canary's Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) for MITRE ATT&CK-mapped attack simulations.

## 📁 Files

- **tests-executed.md** — Log of tests run with results
- **detection-mapping.md** — Attack → Detection lookup table

## 🚀 Installation Reference

On WIN-TARGET (with internet access temporarily):

```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force
Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force
Install-Module -Name invoke-atomicredteam, powershell-yaml -Force
Import-Module invoke-atomicredteam
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics -Force
```

## 📂 Default Atomic Locations

- Atomic tests: `C:\AtomicRedTeam\atomics\`
- PowerShell modules: `C:\Program Files\WindowsPowerShell\Modules\Invoke-AtomicRedTeam\`

## 🧪 Running Tests

```powershell
# See what a test does (no execution)
Invoke-AtomicTest T1082 -ShowDetails

# Run a specific test number
Invoke-AtomicTest T1082 -TestNumbers 1

# Run with cleanup after
Invoke-AtomicTest T1082 -TestNumbers 1
Invoke-AtomicTest T1082 -TestNumbers 1 -Cleanup
```

## ⚠️ Defender Considerations

Windows Defender blocks Mimikatz, Bloodhound, and other known offensive tools used in some Atomic tests. Two approaches:

1. **Use harmless tests** — Discovery techniques (T1082, T1033, T1007) work fine
2. **Temporarily disable Defender** for the test, then re-enable:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Invoke-AtomicTest T1003.001 -TestNumbers 1
Set-MpPreference -DisableRealtimeMonitoring $false
```
