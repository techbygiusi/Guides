# 🛠️ Proxmox Node IP Address Change with Ceph

This guide explains how to change a Proxmox node's IP address, including updates to networking, Corosync, and Ceph configuration, all in one process.

## 🔧 Step-by-Step Instructions

### 1. Update Network Configuration

Edit the network interfaces file to set the new IP address and gateway:

```bash
nano /etc/network/interfaces
```

Edit the hosts file and update any entries with the new IP:

```bash
nano /etc/hosts
```

Change the DNS server if needed:

```bash
nano /etc/resolv.conf
```

### 2. Update Corosync Configuration

Update the Corosync config file and **replace only the IP address of the current node**:

```bash
nano /etc/pve/corosync.conf
```

### 3. Update Known Hosts (if Master Node IP is Changing)

If you are changing the **master node's** IP address, update:

```bash
nano /etc/pve/priv/known_hosts
```

Insert the **new IP** of the master node.

### 4. Update Ceph Configuration (If Using Ceph)

Edit the Ceph configuration to reflect the new networks:

```bash
nano /etc/ceph/ceph.conf
```

Modify these lines:

```ini
cluster_network = 10.10.20.0/24,10.10.0.0/24
public_network  = 10.10.20.0/24,10.10.0.0/24
```

### 5. Reboot the Node

Reboot to apply network and service changes:

```bash
reboot
```

### 6. Remove Old Monitor (MON) via GUI

In the Proxmox web interface, delete the MON of the node you changed.

### 7. Temporarily Edit `ceph.conf` Again

Before recreating the MON, temporarily restrict `public_network`:

```bash
nano /etc/pve/ceph.conf
```

Modify:

```ini
public_network = 10.10.0.0/24
```

### 8. Create the New Monitor (MON)

Use the following command:

```bash
pveceph mon create
```

### 9. Restore Original `ceph.conf`

Edit `ceph.conf` again:

```bash
nano /etc/pve/ceph.conf
```

Restore:

```ini
public_network = 10.10.20.0/24,10.10.0.0/24
```

Make sure the `[mon.<hostname>]` section contains:

```ini
[mon.work]
public_addr = 10.10.0.12
```

> Replace `work` and `10.10.0.12` with your actual hostname and new IP.

### 🔁 Repeat for Other Nodes

Repeat the whole process on any additional nodes (e.g. 3 total).

After completing the change on the last node, you can simplify the network configuration to only:

```ini
public_network  = 10.10.0.0/24
cluster_network = 10.10.0.0/24
```

### 🔄 Final Reboot

Reboot the final node:

```bash
reboot
```

## ✅ Done!

Your Proxmox and Ceph cluster should now be fully updated and operational with the new IP settings.