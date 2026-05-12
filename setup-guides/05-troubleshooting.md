# 05 — Troubleshooting Guide

Every issue documented here is a **real problem** I hit during the actual build. If you're stuck, your fix is probably here.

---

## 🌐 Issue 1: VMnet Subnet Typo

**Symptom:**
- VMs can't ping each other
- `Test-NetConnection` from host shows `DestinationHostUnreachable`
- Host's VMnet10 IP shows something weird like `192.161.10.1`

**Root Cause:**
Single typo when creating the VMware network — `192.161.10.0` instead of `192.168.10.0`.

**Fix:**

```powershell
ipconfig
# Look at "VMware Network Adapter VMnet10" — must show 192.168.10.1
```

If wrong:
1. VMware → Edit → Virtual Network Editor → VMnet10
2. Fix Subnet IP to `192.168.10.0`
3. Apply → OK
4. Disable/Enable the adapter on host:

```powershell
Disable-NetAdapter -Name "VMware Network Adapter VMnet10" -Confirm:$false
Enable-NetAdapter -Name "VMware Network Adapter VMnet10"
```

---

## 📝 Issue 2: inputs.conf Saved as .txt

**Symptom:**
- Forwarder is running but no Sysmon data appears in Splunk
- `Get-ChildItem` of the local folder shows `inputs.conf.txt`

**Root Cause:**
Notepad appends `.txt` to filenames by default unless you select "All Files".

**Fix:**

```powershell
Rename-Item "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf.txt" `
            "C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf"
Restart-Service SplunkForwarder
```

**Prevention:** When saving in Notepad, change **"Save as type"** to **"All Files"** before naming.

---

## 🔒 Issue 3: Sysmon Channel Access Denied (errorCode=5)

**Symptom:**
Forwarder logs show:
```
ERROR - unable to subscribe to Windows Event Log channel 
'Microsoft-Windows-Sysmon/Operational': errorCode=5
```

**Root Cause:**
The SplunkForwarder service doesn't have permission to read the Sysmon Event Log channel.

**Fix:**

```powershell
Add-LocalGroupMember -Group "Event Log Readers" -Member "NETWORK SERVICE"
Add-LocalGroupMember -Group "Event Log Readers" -Member "NT SERVICE\SplunkForwarder"
Restart-Service SplunkForwarder
```

---

## 💾 Issue 4: Splunk Stops Indexing (Disk Space)

**Symptom:**
- Forwarder connection succeeds
- Data is sent (visible in metrics.log: `ev=109`)
- BUT no events appear when searching in Splunk

**Root Cause:**
Splunk pauses indexing when free disk space drops below 5GB (default minFreeSpace).

Check by searching:
```spl
index=_internal source=*splunkd.log* "MinFreeSpace"
```

**Fix (Lab):**

```bash
sudo nano /opt/splunk/etc/system/local/server.conf
```

Add:
```ini
[diskUsage]
minFreeSpace = 2000
```

Save → restart Splunk:
```bash
sudo /opt/splunk/bin/splunk restart
```

**Fix (Proper):** Expand the VM disk in VMware and resize the partition.

---

## 🔍 Issue 5: Sysmon Fields Not Extracting

**Symptom:**
- Events arrive in Splunk
- Raw XML is visible in events
- BUT EventID, Image, CommandLine show blank in tables

**Root Cause:**
Splunk doesn't auto-parse Sysmon XML without an add-on or explicit extraction.

**Fix Option A — Use regex in SPL:**

```spl
index=endpoint sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventID>\d+)</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| rex field=_raw "<Data Name='CommandLine'>(?<CommandLine>[^<]+)</Data>"
| table _time, EventID, Image, CommandLine
```

**Fix Option B — Install Splunk Add-on for Sysmon:**

In Splunk Web UI → Apps → Find More Apps → search "Splunk Add-on for Microsoft Sysmon" → Install → Restart.

---

## 🏷️ Issue 6: Wrong Hostname in Splunk

**Symptom:**
- You renamed the Windows VM to `WIN-TARGET`
- Splunk searches for `host=WIN-TARGET` return nothing
- But `host=DESKTOP-XXXXXX` (the old name) returns data

**Root Cause:**
The forwarder caches the original hostname.

**Fix:**

Create/edit `C:\Program Files\SplunkUniversalForwarder\etc\system\local\server.conf`:

```ini
[general]
serverName = WIN-TARGET
```

Then:
```powershell
Restart-Service SplunkForwarder
```

> ℹ️ Existing events still have the old hostname. Only new events will use the new name.

---

## 🔌 Issue 7: IP Lost After Network Switch

**Symptom:**
- After switching network adapter (NAT → VMnet10 or vice versa)
- `ipconfig` shows `169.254.x.x` (APIPA address)
- Cannot reach other VMs

**Root Cause:**
Windows lost the static IP config when the adapter changed.

**Fix:**

```powershell
# Remove auto-config
Remove-NetIPAddress -InterfaceAlias "Ethernet0" -Confirm:$false

# Re-add static IP
New-NetIPAddress -InterfaceAlias "Ethernet0" `
  -IPAddress 192.168.10.30 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.10.1

Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" `
  -ServerAddresses 192.168.10.20

Restart-Service SplunkForwarder
```

---

## 🚫 Issue 8: Atomic Red Team Blocked by Defender

**Symptom:**
```
Exception calling "Start" with "0" argument(s): "Access is denied"
```

When running tests like `T1059.001` (which uses Mimikatz or Bloodhound).

**Root Cause:**
Windows Defender blocks known offensive tools.

**Fix Option A — Use harmless tests:**
```powershell
Invoke-AtomicTest T1082 -TestNumbers 1   # System Info Discovery (harmless)
Invoke-AtomicTest T1033 -TestNumbers 1   # User Discovery (whoami)
Invoke-AtomicTest T1007 -TestNumbers 1   # Service Discovery
```

**Fix Option B — Disable Defender temporarily:**
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
# Run your test
Set-MpPreference -DisableRealtimeMonitoring $false
```

---

## 🔄 Issue 9: SSH Connection Refused

**Symptom:**
```
ssh: connect to host 192.168.10.10 port 22: Connection refused
```

**Root Cause:**
OpenSSH server isn't running (or isn't installed).

**Fix on Ubuntu:**

```bash
sudo systemctl status ssh

# If inactive:
sudo systemctl start ssh
sudo systemctl enable ssh

# If not installed:
sudo apt update
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```

---

## 🐢 Issue 10: Ubuntu Boot Hangs on "wait for network"

**Symptom:**
At boot, message: `A start job is running for wait for network to be configured (1min / no limit)`

**Root Cause:**
Ubuntu waits for full network setup, including internet routes that don't exist on the isolated lab network.

**Fix:**

```bash
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl mask systemd-networkd-wait-online.service
sudo reboot
```

Boot will be much faster.

---

## 🔁 Issue 11: Forwarder Won't Stop/Restart

**Symptom:**
```
Restart-Service : Service 'SplunkForwarder' cannot be stopped
```

**Fix:**

```powershell
Stop-Service SplunkForwarder -Force
Start-Sleep -Seconds 3
Start-Service SplunkForwarder
Get-Service SplunkForwarder
```

---

## 🧹 Issue 12: General Health Check Commands

When something feels off, run these on each VM to triage:

**Windows Target:**
```powershell
Get-Service SplunkForwarder
Get-Service Sysmon64
Test-NetConnection 192.168.10.10 -Port 9997
ipconfig /all
Get-Content "C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log" -Tail 20
```

**Splunk Server:**
```bash
sudo /opt/splunk/bin/splunk status
sudo ss -tlnp | grep 9997
df -h
sudo tail -f /opt/splunk/var/log/splunk/splunkd.log
```

**Domain Controller:**
```powershell
Get-ADDomain
Get-Service NTDS, DNS, KDC
```
