# 01 — VMware Network Setup

The foundation of the lab. Get this wrong and nothing else will work.

---

## 🎯 Goal

Create an isolated virtual network where all 4 VMs can communicate with each other on the `192.168.10.0/24` subnet.

---

## Step 1 — Open Virtual Network Editor

1. Launch VMware Workstation
2. Go to **Edit → Virtual Network Editor**
3. If prompted, click **Change Settings** for admin privileges

---

## Step 2 — Create VMnet10

1. Click **Add Network** → select **VMnet10** → OK
2. Configure it as follows:

| Setting | Value |
|---|---|
| Type | **Host-only** |
| Subnet IP | `192.168.10.0` |
| Subnet mask | `255.255.255.0` |
| Use local DHCP service | ❌ **Unchecked** |
| Connect host to this network | ✅ Checked |

3. Click **Apply** → **OK**

---

## Step 3 — Verify on Host PC

Open PowerShell on your host and run:

```powershell
ipconfig
```

Look for **"VMware Network Adapter VMnet10"** with IP `192.168.10.1`:

```
Ethernet adapter VMware Network Adapter VMnet10:
   IPv4 Address. . . . . . . . . . . : 192.168.10.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
```

> ⚠️ **CRITICAL CHECK:** Verify the IP says `192.168.10.1` and NOT `192.161.10.1`. A common typo in VMware breaks the entire lab.

---

## Step 4 — Configure Each VM's Network Adapter

For every VM you create:

1. Right-click VM → **Settings**
2. Select **Network Adapter**
3. Choose **Custom: Specific virtual network**
4. Select **VMnet10** from the dropdown
5. Click **OK**

---

## ✅ Verification Checklist

- [ ] VMnet10 exists in Virtual Network Editor
- [ ] Subnet is `192.168.10.0/24`
- [ ] Host adapter shows IP `192.168.10.1`
- [ ] All VMs have Network Adapter set to **Custom: VMnet10**

---

## Common Issues

**Problem:** Host adapter shows `192.161.10.1` instead of `192.168.10.1`
**Fix:** Open Virtual Network Editor → VMnet10 → Fix subnet IP to `192.168.10.0` → Apply

**Problem:** VMs can't ping each other
**Fix:** Confirm all VMs are set to VMnet10 (not NAT, not Bridged)

**Problem:** Host can't reach VMs
**Fix:** Disable and re-enable the VMnet10 adapter in Windows:
```powershell
Disable-NetAdapter -Name "VMware Network Adapter VMnet10" -Confirm:$false
Enable-NetAdapter -Name "VMware Network Adapter VMnet10"
```
