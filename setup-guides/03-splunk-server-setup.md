# 03 — Splunk Server Setup (Ubuntu 22.04)

Install the SIEM brain of your SOC.

---

## 🎯 Goal

Set up Splunk Enterprise on Ubuntu to receive logs from all endpoint VMs and provide a web UI for searching, alerting, and dashboards.

---

## Step 1 — Create the VM in VMware

1. **Create a New Virtual Machine** → "I will install the operating system later"
2. Guest OS: **Linux → Ubuntu 64-bit**
3. VM Name: `SPLUNK-SERVER`
4. Disk: **60 GB minimum** (Splunk needs space)
5. Customize Hardware:

| Setting | Value |
|---|---|
| RAM | 4–8 GB |
| CPU | 2–4 cores |
| Network Adapter | **Custom: VMnet10** |
| CD/DVD | Browse to Ubuntu 22.04 LTS Server ISO |

---

## Step 2 — Install Ubuntu Server

During installation:

| Field | Value |
|---|---|
| Server name | `splunkserver` |
| Username | `splunk` |
| Password | (your choice) |
| **OpenSSH server** | ✅ **Install** |
| Featured snaps | ❌ Skip all |

---

## Step 3 — Configure Static IP

After first login, find your network interface name:

```bash
ip a
# Look for ens33, ens32, or similar
```

Edit netplan config (replace `ens33` with your interface):

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Replace contents with:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.10.10/24
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Apply changes:

```bash
sudo netplan apply
ip a  # Verify 192.168.10.10 is assigned
```

---

## Step 4 — SSH from Host PC

From now on, work via SSH instead of the VMware console window:

```powershell
ssh splunk@192.168.10.10
```

---

## Step 5 — Transfer & Install Splunk Enterprise

On host PC, download Splunk Enterprise `.deb` from splunk.com.

Transfer it to the VM:

```powershell
scp "C:\Users\YourName\Downloads\splunk-*.deb" splunk@192.168.10.10:~
```

On the VM:

```bash
# Install Splunk
sudo dpkg -i splunk-*.deb

# Start Splunk for the first time
cd /opt/splunk/bin
sudo ./splunk start --accept-license

# When prompted:
# Admin username: admin
# Password: Admin@1234 (or your choice, min 8 chars)
```

---

## Step 6 — Enable Auto-Start, Receiving Port, and Create Index

```bash
# Auto-start on boot
sudo /opt/splunk/bin/splunk enable boot-start -user splunk --accept-license --answer-yes

# Listen for forwarder connections
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:Admin@1234

# Create the endpoint index
sudo /opt/splunk/bin/splunk add index endpoint -auth admin:Admin@1234
```

---

## Step 7 — Configure Firewall

```bash
sudo ufw allow 8000/tcp    # Splunk Web UI
sudo ufw allow 9997/tcp    # Forwarder receiver
sudo ufw allow 22/tcp      # SSH
sudo ufw enable
sudo ufw status
```

---

## Step 8 — Lower Disk Space Threshold (Lab-Only Tweak)

By default, Splunk stops indexing if free space < 5GB. For a lab, lower this:

```bash
sudo nano /opt/splunk/etc/system/local/server.conf
```

Add at the end:

```ini
[diskUsage]
minFreeSpace = 2000
```

Save (Ctrl+X → Y → Enter), then:

```bash
sudo /opt/splunk/bin/splunk restart
```

---

## Step 9 — Access the Web UI

From your **host PC** browser:

```
http://192.168.10.10:8000
```

Login: `admin` / `Admin@1234`

You should see the Splunk home dashboard.

---

## ✅ Verification Checklist

- [ ] SSH works from host: `ssh splunk@192.168.10.10`
- [ ] Web UI loads at `http://192.168.10.10:8000`
- [ ] `splunk status` shows running
- [ ] Port 9997 listening: `sudo ss -tlnp | grep 9997`
- [ ] Index `endpoint` exists in Settings → Indexes

---

## ⚠️ Critical Notes

- **Disk space matters** — Splunk will pause indexing if low
- **Default credentials** — change `Admin@1234` to something secure in real use
- **Free license** — after 60 days, ingestion caps at 500 MB/day (plenty for a lab)
