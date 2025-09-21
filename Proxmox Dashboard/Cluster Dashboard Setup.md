# 📊 Proxmox Cluster Dashboard Setup

This document describes step by step how we built a **Proxmox dashboard** on a fresh **Debian LXC** — including **Proxmox user & API token, Nginx, Filebrowser, Node.js proxy, and HTML frontend**.

<img width="1049" height="628" alt="image" src="https://github.com/user-attachments/assets/66fc5b2e-32c3-479c-9791-20c8b34bcdd4" />

---

## 1. Base: Debian LXC

We started with a **Debian LXC container**, serving as the base for:
- Webserver (**Nginx**)  
- File management (**Filebrowser**)  
- Proxy service (**Node.js**)  
- Dashboard HTML files  

---

## 2. Proxmox User & API Token

To allow the dashboard to access cluster data via the API, we created a dedicated **user + role + token** in Proxmox.

### Steps

```bash
# Create a new role with minimal read-only privileges
pveum roleadd DashboardReader -privs "Sys.Audit VM.Audit Datastore.Audit Node.Audit"

# Create user (no login, API-only)
pveum useradd dashboard@pve

# Create API token
pveum user token add dashboard@pve dashboard

# Assign permissions at the cluster level
pveum aclmod / -token 'dashboard@pve!dashboard' -role DashboardReader

# Optionally restrict to specific nodes
pveum aclmod /nodes/alex  -token 'dashboard@pve!dashboard' -role DashboardReader
pveum aclmod /nodes/haley -token 'dashboard@pve!dashboard' -role DashboardReader
pveum aclmod /nodes/luke  -token 'dashboard@pve!dashboard' -role DashboardReader
```

👉 Result:  
- User: `dashboard@pve`  
- Token: `dashboard`  
- Role: `DashboardReader` (read-only audit permissions)  

The **API token string** looks like:  
```
PVEAPIToken=dashboard@pve!dashboard=xxxx-xxxx-xxxx
```

---

## 3. Nginx Installation

Update repositories and install Nginx:
```bash
apt-get update
apt-get install nginx
```

Verify installation:
```
nginx -v
```

Check container IP (if needed):
```
ip a
```

The default Nginx web root is `/var/www/html`, accessible at:
```
http://[your-ip-address]
```

---

## 4. Filebrowser Setup

To manage web root files conveniently in the browser, we installed **Filebrowser**.

Install curl:
```
apt install curl
```

Download and install Filebrowser:
```
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
```

---

### Configuring Filebrowser
Create configuration:
```bash
cd /etc/
nano filebrowser.json
```

Paste:
```json
{
  "port": 8080,
  "baseURL": "/",
  "address": "0.0.0.0",
  "log": "stdout",
  "database": "/etc/filebrowser.db",
  "root": "/var/www/html/"
}
```

---

### Systemd Service for Filebrowser
Create service:
```
nano /etc/systemd/system/filebrowser.service
```

Add:
```ini
[Unit]
Description=File Browser
After=network.target

[Service]
User=www-data
Group=www-data
ExecStart=/usr/local/bin/filebrowser -c /etc/filebrowser.json

[Install]
WantedBy=multi-user.target
```

👉 Running Filebrowser as `www-data` ensures both **nginx** and **Filebrowser** share access to `/var/www/html`.

---

### Enable & Start Filebrowser
```bash
systemctl enable filebrowser.service
systemctl start filebrowser.service
```

Now available at:
```
http://[your-ip-address]:8080
```

---

### Default Credentials
- **Username:** `admin`  
- **Password:** `admin`  

Reset password if needed:
```
systemctl stop filebrowser.service
```
```
filebrowser -d /etc/filebrowser.db users update admin --password NEWPASS
```
```
systemctl start filebrowser.service
```

---

## 5. Proxy Service (Node.js)

### Why a Proxy?
- Browser cannot directly call Proxmox API (CORS, HTTPS, auth issues).  
- Proxy acts as a **secure middleman**:  
  - adds API token  
  - handles HTTPS and CORS  
  - returns JSON  

### Installation
```bash
apt update && apt install -y nodejs npm
mkdir -p /opt/proxmox-dashboard
cd /opt/proxmox-dashboard
npm init -y
npm install express node-fetch@2
```

### proxy.js
```js
const express = require("express");
const fetch = require("node-fetch");
const app = express();

const TOKEN = "PVEAPIToken=dashboard@pve!dashboard=SECRET";
const API_URL = "https://[your-cluster-node-ip-or-domain]/api2/json";

// Allow CORS
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "GET, OPTIONS");
  res.header("Access-Control-Allow-Headers", "Authorization, Content-Type");
  if (req.method === "OPTIONS") return res.sendStatus(200);
  next();
});

// Proxy handler
app.use(/^\/api\/(.*)/, async (req, res) => {
  try {
    const targetPath = req.params[0] || "";
    const queryString = new URLSearchParams(req.query).toString();
    const targetUrl = `${API_URL}/${targetPath}${queryString ? "?" + queryString : ""}`;

    console.log("Proxying to:", targetUrl);

    const response = await fetch(targetUrl, { headers: { Authorization: TOKEN } });
    const data = await response.text();

    res.status(response.status).send(data);
  } catch (err) {
    console.error("Proxy error:", err);
    res.status(500).send({ error: "Proxy failed" });
  }
});

app.listen(3000, () => console.log("Proxy running on port 3000"));
```

### Systemd Unit
```
nano /etc/systemd/system/proxmox-proxy.service
```
```ini
[Unit]
Description=Proxmox Proxy
After=network.target

[Service]
ExecStart=/usr/bin/node /opt/proxmox-dashboard/proxy.js
Restart=always
User=root
WorkingDirectory=/opt/proxmox-dashboard

[Install]
WantedBy=multi-user.target
```

Enable:
```bash
systemctl daemon-reload
systemctl enable proxmox-proxy
systemctl start proxmox-proxy
```

---

## 6. Dashboard Frontend
```
nano /var/www/html/index.html
```
### Features
- Shows **CPU, RAM, Disk** with progress bars & color coding (green/yellow/red)  
- Displays **IP address** of each node  
- **Status** (online/offline, with colors)  
- **Uptime** formatted as `Xd Xh Xm Xs`  
- Auto-refresh every **5 seconds**  

```bash
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Proxmox Dashboard</title>
  <meta name="viewport" content="width=800, height=480, initial-scale=1.0">

  <style>
    body {
      margin: 0;
      width: 780px;
      height: 480px;
      background: #0e0e0e;
      color: #e0e0e0;
      font-family: "Segoe UI", sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: flex-start;
      padding: 10px;
      overflow: hidden;
    }

    h2 {
      margin: 5px 0 20px;
      font-size: 24px;
      font-weight: 700;
      background: linear-gradient(90deg, #00ffc8, #00aaff);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    .nodes {
      display: flex;
      gap: 20px;
      justify-content: center;
      align-items: stretch;
      flex: 1;
      width: 100%;
      padding-bottom: 20px;
      box-sizing: border-box;
    }

    .node-card {
      flex: 0 1 240px;
      min-width: 220px;
      max-width: 260px;
      background: linear-gradient(180deg, #1c1c1c, #161616);
      border-radius: 20px;
      padding: 20px 15px;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
    }

    .node-title {
      font-size: 20px;
      font-weight: 700;
      margin-bottom: 15px;
      color: #00e0ff;
      letter-spacing: 0.5px;
    }

    .metric {
      font-size: 12px;
      margin: 6px 0;
      text-align: center;
    }

    .status-online {
      color: #00ff95;
      font-weight: 600;
    }

    .status-offline {
      color: #ff5555;
      font-weight: 600;
    }

    .progress {
      width: 100%;
      height: 10px;
      border-radius: 5px;
      background: #2a2a2a;
      overflow: hidden;
      margin: 4px 0 10px 0;
    }

    .progress-bar {
      height: 100%;
      transition: width 0.5s ease, background 0.5s ease;
    }
  </style>
</head>
<body>
  <h2>Tiny Lab</h2>
  <div class="nodes" id="nodes">Lade Nodes…</div>

<script>
const apiURL = "https://[your-proxy-ip-or-domain]/api";

function formatUptime(seconds) {
  if (!seconds) return "0s";
  let d = Math.floor(seconds / 86400);
  let h = Math.floor((seconds % 86400) / 3600);
  let m = Math.floor((seconds % 3600) / 60);
  let s = seconds % 60;
  return [d ? d + "d" : "", h ? h + "h" : "", m ? m + "m" : "", s ? s + "s" : ""]
    .filter(Boolean)
    .join(" ");
}

async function safeFetch(url) {
  try {
    let res = await fetch(url);
    if (!res.ok) throw new Error("HTTP " + res.status);
    return await res.json();
  } catch {
    return { data: [] };
  }
}

function getBarColor(percent) {
  if (percent < 60) return "linear-gradient(90deg, #00ffc8, #00aaff)";
  if (percent < 85) return "linear-gradient(90deg, #ffaa00, #ff8800)";
  return "linear-gradient(90deg, #ff4444, #cc0000)";
}

function formatBytesPerSec(val) {
  if (val > 1e6) return (val/1e6).toFixed(2) + " MB/s";
  if (val > 1e3) return (val/1e3).toFixed(2) + " KB/s";
  return val.toFixed(2) + " B/s";
}

async function loadCluster() {
  try {
    let res = await fetch(apiURL + "/cluster/resources?type=node");
    let data = await res.json();
    let nodes = data.data.filter(r => r.type === "node");
    nodes.sort((a, b) => a.node.localeCompare(b.node));

    let html = "";

    for (let n of nodes) {
      try {
        let reachable = n.status === "online";
        let cpuPerc = 0, cpuTotal = n.maxcpu || 0;
        let memText = "-", diskText = "-", netText = "↓ 0 B/s ↑ 0 B/s";
        let memPerc = 0, diskPerc = 0;
        let vms = 0, lxc = 0;
        let ip = "-";

        if (reachable) {
          let rrd = await safeFetch(apiURL + `/nodes/${n.node}/rrddata?timeframe=hour`);
          let samples = Array.isArray(rrd.data) ? rrd.data : [];

          // CPU
          cpuPerc = n.cpu ? (n.cpu * 100).toFixed(1) : 0;

          // RAM
          let memUsed = n.mem || 0;
          let memTotal = n.maxmem || 1;
          memPerc = ((memUsed / memTotal) * 100).toFixed(1);
          memText = `${memPerc}% (${(memUsed/1073741824).toFixed(1)} GB / ${(memTotal/1073741824).toFixed(1)} GB)`;

          // Disk
          let diskUsed = n.disk || 0;
          let diskTotal = n.maxdisk || 1;
          diskPerc = ((diskUsed / diskTotal) * 100).toFixed(1);
          diskText = `${diskPerc}% (${(diskUsed/1073741824).toFixed(1)} GB / ${(diskTotal/1073741824).toFixed(1)} GB)`;

          // VMs and CTs
          let vmRes = await fetch(apiURL + `/nodes/${n.node}/qemu`);
          let vmData = await vmRes.json();
          let lxcRes = await fetch(apiURL + `/nodes/${n.node}/lxc`);
          let lxcData = await lxcRes.json();
        
          vms = vmData?.data?.length || 0;
          lxc = lxcData?.data?.length || 0;

          // IP
          let netData = await safeFetch(apiURL + `/nodes/${n.node}/network`);
          if (Array.isArray(netData.data)) {
            let ips = netData.data
              .filter(i => i?.address)
              .map(i => i.address)
              .filter(a => a && !a.startsWith("127.") && !a.startsWith("169.254"));
            ip = ips[0] || "-";
          }
        }

        let uptime = reachable ? formatUptime(n.uptime) : "0s";
        let status = reachable ? "online" : "offline";

        html += `
          <div class="node-card">
            <div class="node-title">${n.node}</div>
            <div class="metric">CPU: ${cpuPerc}% von ${cpuTotal}</div>
            <div class="progress"><div class="progress-bar" style="width:${cpuPerc}%; background:${getBarColor(cpuPerc)}"></div></div>
            <div class="metric">RAM: ${memText}</div>
            <div class="progress"><div class="progress-bar" style="width:${memPerc}%; background:${getBarColor(memPerc)}"></div></div>
            <div class="metric">Disk: ${diskText}</div>
            <div class="progress"><div class="progress-bar" style="width:${diskPerc}%; background:${getBarColor(diskPerc)}"></div></div>
            <div class="metric">VMs: ${vms} | LXCs: ${lxc}</div>
            <div class="metric">IP: ${ip}</div>
            <div class="metric">Status: <span class="status-${status}">${status}</span></div>
            <div class="metric">Uptime: ${uptime}</div>
          </div>
        `;
      } catch (e) {
        console.error("Fehler bei Node:", n.node, e);
      }
    }

    document.getElementById("nodes").innerHTML = html;
  } catch (e) {
    document.getElementById("nodes").innerText = "Fehler beim Laden!";
  }
}

loadCluster();
setInterval(loadCluster, 5000);
</script>
</body>
</html>
```

---

## 7. Result

✅ Dashboard accessible at `https://[your-nginx-ip-or-domain]`  
✅ Proxy hides token and handles HTTPS/CORS  
✅ Nginx + Filebrowser simplify file management  
✅ Proxy autostarts via systemd  
✅ Clean node cards: **CPU, RAM, Disk, IP, Status, Uptime**  

---

# ✅ Summary
1. Deployed a fresh Debian LXC  
2. Created **Proxmox user + token** with `DashboardReader` role (read-only)  
3. Installed **Nginx + Filebrowser** for web file management  
4. Built a **Node.js proxy service** with systemd autostart  
5. Developed a **modern HTML dashboard** for live cluster stats  
