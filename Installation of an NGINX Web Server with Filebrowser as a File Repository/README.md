# Installation of an NGINX Web Server with Filebrowser as a File Repository on Proxmox

## Getting Started

To begin, you need a properly set up Proxmox system. In this guide, I am using a basic setup. Proxmox has been freshly installed and cleaned up using the [Proxmox VE Post Install](https://proxmoxve-scripts.com/scripts?id=Proxmox%20VE%20Post%20Install) script from Proxmox Helper Scripts.

Next, create a new LXC container in the Proxmox web interface. Once the container is created, we can begin installing the NGINX web server.

Resources:

- [NGINX](https://nginx.org/en/)
- [Filebrowser](https://filebrowser.org/)

## Setup

First, update the package repositories by running:

```bash
apt-get update
```

Then, install NGINX with the following command:

```bash
apt-get install nginx
```

You can verify the installation by checking the version with:

```bash
nginx -v
```

If you're unsure of the container's IP address, you can find it by typing:

```bash
ip a
```

After this, the website should be accessible at `http://[your-ip-address]`.

Now, to manage website data, we will install Filebrowser for easy file access. NGINX's default data directory is located at `/var/www/html`.

### Installing Filebrowser

First, install `curl`:

```bash
apt install curl
```

Then, install Filebrowser with the following command:

```bash
curl -fsSL https://raw.githubusercontent.com/filebrowser/get/master/get.sh | bash
```

### Configuring Filebrowser

Now, point Filebrowser to the NGINX directory and configure it as a service. First, create a configuration file for Filebrowser:

```bash
cd /etc/
nano filebrowser.json
```

In the file, paste the following content:

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

Save the file by pressing `CTRL + X`, then `Y`, and `ENTER`.

Next, create a systemd service file for Filebrowser:

```bash
nano /etc/systemd/system/filebrowser.service
```

In the file, add the following:

```ini
[Unit]
Description=File Browser
After=network.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/filebrowser -c /etc/filebrowser.json

[Install]
WantedBy=multi-user.target
```

### Enabling and Starting Filebrowser

Enable the Filebrowser service to start on boot and start it with the following commands:

```bash
systemctl enable filebrowser.service
systemctl start filebrowser.service
```

Filebrowser will now be accessible at `http://[your-ip-address]:8080`.

The default username and password are `admin` / `admin`. These credentials can be changed through the Filebrowser web interface.

---

**Recommendation:** It is highly recommended to secure both the website and Filebrowser with a TLS certificate to protect your data from being intercepted.
