# ARM Ripper Update Cycle

## 📁 Directory Structure

```
/home/giuseppe/
├── run-arm-rippers.sh         # Start script (active)
├── arm-rippers-config.json    # Configuration (active)
├── docker-setup.sh            # Original setup script (ARM project)
└── backups/
    └── YYYYMMDD/              # Archive per update date
        ├── run-arm-rippers.sh.bak
        ├── arm-rippers-config.json.bak-YYYYMMDD
        ├── arm-rippers-inspect-YYYYMMDD.json
        └── docker-setup.sh.bak

/home/arm/
├── config/                    # ARM configuration (volume)
├── logs/                      # Logs (volume)
├── media/                     # Media (volume)
└── music/                     # Music (volume)

/mnt/jellyfin-freigabe/        # Jellyfin share (volume)
```

## 🔍 Status & Logs

```bash
# Check container status
sudo docker ps

# Last 50 log lines
sudo docker logs arm-rippers --tail 50

# Follow logs live
sudo docker logs arm-rippers -f

# Container details
sudo docker inspect arm-rippers
```

## ▶️ Start / Stop Container

```bash
# Start (via script)
sudo bash run-arm-rippers.sh

# Stop
sudo docker stop arm-rippers

# Stop + remove
sudo docker stop arm-rippers && sudo docker rm arm-rippers

# Restart
sudo docker restart arm-rippers
```

## 🔄 Update Procedure

### 1. Create Backup

```bash
# Save old image (before overwriting!)
sudo docker tag arm-rippers:backup-2025-08-30 arm-rippers:pre-update-$(date +%Y%m%d)

# Back up scripts & config
sudo cp run-arm-rippers.sh run-arm-rippers.sh.bak
sudo cp arm-rippers-config.json arm-rippers-config.json.bak-$(date +%Y%m%d)
sudo docker inspect arm-rippers > arm-rippers-inspect-$(date +%Y%m%d).json
sudo cp -r /home/arm/config /home/arm/config.bak-$(date +%Y%m%d)
```

### 2. Pull New Image

```bash
sudo docker pull automaticrippingmachine/automatic-ripping-machine:latest
```

### 3. Re-tag (run script stays unchanged)

```bash
sudo docker tag automaticrippingmachine/automatic-ripping-machine:latest arm-rippers:backup-2025-08-30
```

### 4. Restart Container

```bash
sudo docker stop arm-rippers
sudo docker rm arm-rippers
sudo bash run-arm-rippers.sh
```

### 5. Verify

```bash
sudo docker ps
sudo docker logs arm-rippers --tail 50
```

## ⏪ Rollback

```bash
sudo docker stop arm-rippers && sudo docker rm arm-rippers
sudo docker tag arm-rippers:pre-update-YYYYMMDD arm-rippers:backup-2025-08-30
sudo bash run-arm-rippers.sh
```

> **Adjust date:** `YYYYMMDD` → e.g. `20260406`

## 🗂️ Clean Up Backup Archive

```bash
# Create backup folder with today's date
sudo mkdir -p ~/backups/$(date +%Y%m%d)

# Move all .bak and inspect files
sudo mv ~/arm-rippers-config.json.bak-* ~/backups/$(date +%Y%m%d)/
sudo mv ~/arm-rippers-inspect-*.json ~/backups/$(date +%Y%m%d)/
sudo mv ~/docker-setup.sh.bak ~/backups/$(date +%Y%m%d)/
sudo mv ~/run-arm-rippers.sh.bak ~/backups/$(date +%Y%m%d)/

# Verify
ls ~/backups/$(date +%Y%m%d)/
```

## 🐳 Docker Image Management

```bash
# List all local images
sudo docker images

# Remove unused images
sudo docker image prune

# Remove all unused resources (images, containers, volumes, networks)
sudo docker system prune

# Show disk usage
sudo docker system df

# Delete a specific image
sudo docker rmi arm-rippers:pre-update-YYYYMMDD
```

## ⚙️ Container Configuration (Reference)

```bash
docker run \
  --name=arm-rippers \
  --hostname=f945c8a15cb5 \
  --mac-address=2e:57:52:d9:78:14 \
  --cpuset-cpus=2-7 \
  --volume /home/arm/logs:/home/arm/logs \
  --volume /home/arm/media:/home/arm/media \
  --volume /home/arm/music:/home/arm/music \
  --volume /mnt/jellyfin-freigabe:/mnt/jellyfin-freigabe \
  --volume /home/arm/config:/etc/arm/config \
  --volume /home/arm:/home/arm \
  --env=ARM_UID=1001 \
  --env=ARM_GID=1001 \
  --network=bridge \
  --privileged \
  --workdir=/home/arm \
  -p 8080:8080 \
  --restart=always \
  --device /dev/sr0:/dev/sr0 \
  --device /dev/dri/renderD128:/dev/dri/renderD128 \
  --device /dev/dri:/dev/dri \
  --runtime=runc \
  -d arm-rippers:backup-2025-08-30 /sbin/my_init
```

## 🌐 Web UI

```
http://<server-ip>:8080
```

## ⚠️ Known Notes

| Message | Meaning | Action |
|---|---|---|
| `Database is not current, update required` | DB schema outdated after update | Triggered automatically on next job, or trigger manually |
| `Intel QuickSync supported!` | Hardware encoding active | ✅ OK |
| `Handbrake call successful` | HandBrake working | ✅ OK |
