# 04 — Windows 11 Target Setup

The victim endpoint where attacks happen and where deep telemetry is generated.

---

## 🎯 Goal

Deploy Windows 11 Pro, join the domain, install Sysmon, install the Splunk Universal Forwarder, and confirm logs reach Splunk.

---

## Step 1 — Create the VM

1. **Create a New Virtual Machine** → "I will install the operating system later"
2. Guest OS: **Microsoft Windows → Windows 11 x64**
3. VM Name: `WIN-TARGET`
4. Disk: **50 GB**
5. Customize Hardware:

| Setting | Value |
|---|---|
| RAM | 2–4 GB |
| CPU | 2 cores |
| Network Adapter | **Custom: VMnet10** |
| CD/DVD | Windows 11 ISO |

---

## Step 2 — Install Windows 11 Pro

### Bypass the Network Requirement

When you reach "Let's connect you to a network":

1. Press **Shift + F10** → opens command prompt
2. Type: `OOBE\BYPASSNRO` → Enter
3. Windows reboots, and "I don't have internet" option becomes available

### Continue Setup

- Choose **"Windows 11 Pro"** when prompted (NOT Home — Home blocks RDP, domain join, GPO)
- Click **"I don't have a product key"** — runs unactivated for lab use
- Create a local account: `labuser` with password `Lab@1234`
- Turn all privacy settings **Off**

### After First Boot

Install **VMware Tools**:
- VMware menu → VM → Install VMware Tools
- Run setup64.exe from the inserted CD
- Restart

---

## Step 3 — Set Static IP

Open **PowerShell as Administrator** (right-click Start → Windows Terminal Admin):

```powershell
# Find network adapter name
Get-NetAdapter

# Set static IP (use your actual adapter name)
New-NetIPAddress -InterfaceAlias "Ethernet0" `
  -IPAddress 192.168.10.30 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.10.1

# Set DNS to Domain Controller
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" `
  -ServerAddresses 192.168.10.20

# Verify
ipconfig /all
```

Confirm output shows:
```
IPv4 Address: 192.168.10.30
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
DNS Servers: 192.168.10.20
```

---

## Step 4 — Join Domain

**DC01 must be running first.**

```powershell
# Rename and join domain
Rename-Computer -NewName "WIN-TARGET" -Force
Add-Computer -DomainName "soc.local" -Credential soc\Administrator -Restart
```

After reboot, log in as `soc\Administrator`.

---

## Step 5 — Enable Advanced Audit Policies

Open PowerShell as Admin:

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
auditpol /set /subcategory:"Sensitive Privilege Use" /success:enable /failure:enable

# Enable command line logging in process events
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f

# Verify
auditpol /get /category:*
```

---

## Step 6 — Enable RDP (for Brute Force Simulation)

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' `
  -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

---

## Step 7 — Install Sysmon

### Get Sysmon (temporarily switch to NAT for internet, or download on host and copy)

Download:
- **Sysmon:** https://download.sysinternals.com/files/Sysmon.zip
- **Config:** https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml

Place both files in `C:\Sysmon\` on the VM.

### Install

```powershell
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml

# Verify
Get-Service Sysmon64
```

Should show: `Status: Running`

> ℹ️ If you forget the XML, run `.\Sysmon64.exe -accepteula -i` (uses defaults — still works for the lab).

---

## Step 8 — Install Splunk Universal Forwarder

Download the `.msi` from splunk.com and copy to the VM.

During install:

| Field | Value |
|---|---|
| Username | admin |
| Password | Admin@1234 |
| **Receiving Indexer** | 192.168.10.10 |
| **Port** | 9997 |
| Deployment Server | ❌ Leave blank |

---

## Step 9 — Configure Forwarder

Create two config files. **Save as "All Files" in Notepad, NOT .txt!**

### inputs.conf

Path: `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```ini
[WinEventLog://Security]
disabled = 0
index = endpoint
renderXml = true

[WinEventLog://System]
disabled = 0
index = endpoint
renderXml = true

[WinEventLog://Application]
disabled = 0
index = endpoint
renderXml = true

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = endpoint
renderXml = true
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

### outputs.conf

Path: `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`

```ini
[tcpout]
defaultGroup = splunk_indexer

[tcpout:splunk_indexer]
server = 192.168.10.10:9997
```

### Grant Permissions

```powershell
Add-LocalGroupMember -Group "Event Log Readers" -Member "NT SERVICE\SplunkForwarder"
```

### Restart Forwarder

```powershell
Restart-Service SplunkForwarder
Get-Service SplunkForwarder
```

Should show: `Status: Running`

---

## Step 10 — Verify Data in Splunk

Wait 2–3 minutes, then in Splunk Web UI search:

```spl
index=endpoint host=DESKTOP-*
| stats count by sourcetype
```

You should see all four sourcetypes:
- `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`
- `XmlWinEventLog:Security`
- `XmlWinEventLog:System`
- `XmlWinEventLog:Application`

---

## ✅ Verification Checklist

- [ ] Static IP `192.168.10.30` set correctly
- [ ] Domain join successful (login as `soc\Administrator` works)
- [ ] Audit policies enabled
- [ ] Sysmon service running
- [ ] Splunk Forwarder service running
- [ ] inputs.conf and outputs.conf (NOT .txt) exist
- [ ] Splunk shows events from this host

---

## ⚠️ Common Pitfalls

1. **Notepad saves as .txt** — set "Save as type" to **All Files** before saving config files
2. **Hostname caching** — the forwarder uses the original Windows hostname (DESKTOP-XXXXXX), not what you renamed to. Search using both.
3. **Sysmon access denied (errorCode=5)** — fix with `Add-LocalGroupMember -Group "Event Log Readers" -Member "NT SERVICE\SplunkForwarder"`
4. **No internet** — for downloading Sysmon/Forwarder, temporarily switch network to NAT, then back to VMnet10
