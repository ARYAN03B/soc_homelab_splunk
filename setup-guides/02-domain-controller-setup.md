# 02 — Domain Controller Setup (Windows Server 2022)

Build the Active Directory backbone of the lab.

---

## 🎯 Goal

Create a Windows Server 2022 VM and promote it to a Domain Controller for the `soc.local` domain.

---

## Step 1 — Create the VM in VMware

1. **Create a New Virtual Machine** → "I will install the operating system later"
2. Guest OS: **Microsoft Windows → Windows Server 2022**
3. VM Name: `DC01`
4. Disk: **50 GB** (single file)
5. Customize Hardware:

| Setting | Value |
|---|---|
| RAM | 2–4 GB |
| CPU | 2 cores |
| Network Adapter | **Custom: VMnet10** |
| CD/DVD | Browse to Windows Server 2022 ISO |

6. Click **Finish** → Power on

---

## Step 2 — Install Windows Server 2022

1. Boot from ISO → choose **Windows Server 2022 Standard Evaluation**
2. Choose **Custom install** → select the entire disk
3. Wait for installation (~10 minutes)
4. Set the **Administrator password** when prompted

> ℹ️ If you installed **Server Core** (no GUI), you'll see the `sconfig` menu. That's fine — Active Directory works on Server Core.

---

## Step 3 — Configure Network and Computer Name

From `sconfig` menu, press **15** → Enter → type `powershell` → Enter

Run these commands:

```powershell
# Rename the computer
Rename-Computer -NewName "DC01" -Force

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet0" `
  -IPAddress 192.168.10.20 `
  -PrefixLength 24 `
  -DefaultGateway 192.168.10.1

# Set DNS to itself (will be DNS server after AD install)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" `
  -ServerAddresses 127.0.0.1

# Restart to apply name change
Restart-Computer -Force
```

---

## Step 4 — Install Active Directory Domain Services

After restart, return to `sconfig` → 15 → `powershell`, then:

```powershell
# Install the AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
  -DomainName "soc.local" `
  -DomainNetbiosName "SOC" `
  -InstallDns `
  -SafeModeAdministratorPassword (ConvertTo-SecureString "Admin@1234!" -AsPlainText -Force) `
  -Force
```

The server will reboot automatically after ~3 minutes.

---

## Step 5 — Verify AD is Working

After reboot, log back in and run:

```powershell
Get-ADDomain | Select DNSRoot, NetBIOSName, DomainMode
```

You should see:

```
DNSRoot     : soc.local
NetBIOSName : SOC
DomainMode  : Windows2016Domain
```

---

## Step 6 — Create Test Users

```powershell
# Create a standard test user
New-ADUser -Name "John Smith" -SamAccountName "jsmith" `
  -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

# Create another test user
New-ADUser -Name "Test User" -SamAccountName "tuser" `
  -AccountPassword (ConvertTo-SecureString "Welcome1!" -AsPlainText -Force) `
  -Enabled $true -PasswordNeverExpires $true

# Verify
Get-ADUser -Filter * | Select Name, SamAccountName
```

---

## ✅ Verification Checklist

- [ ] DC01 reaches 192.168.10.20 (verify with `ipconfig`)
- [ ] `Get-ADDomain` returns `soc.local`
- [ ] Test users `jsmith` and `tuser` exist
- [ ] DNS service is running (`Get-Service DNS`)

---

## ⚠️ Critical Notes

- **Use weak passwords intentionally** (Password123!, Welcome1!) — these become brute force targets later
- **Never reuse these credentials** anywhere outside the lab
- The DC is the **single most important VM** — boot it first every time
