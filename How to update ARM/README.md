# ARM Ripper Administration

## Directory Structure

```text
/home/giuseppe/
├── run-arm-rippers.sh
├── arm-rippers-config.json
├── docker-setup.sh
└── backups/

/home/arm/
├── config/
├── logs/
├── media/
├── music/
└── start_arm_container.sh

/mnt/jellyfin-freigabe/
```

## Current Setup

ARM runs as a Docker container using the official image:

```text
automaticrippingmachine/automatic-ripping-machine:latest
```

Current ARM version:

```text
2.24.3
```

The container is managed by:

```text
arm-rippers.service
```

The service starts:

```text
/home/arm/start_arm_container.sh
```

The startup script waits until `/mnt/jellyfin-freigabe` is mounted before starting the container.

The ARM web interface is available on:

```text
http://<server-ip>:8080
```

## Service Management

Check service status:

```bash
sudo systemctl status arm-rippers.service
```

Restart service:

```bash
sudo systemctl restart arm-rippers.service
```

Stop service:

```bash
sudo systemctl stop arm-rippers.service
```

Start service:

```bash
sudo systemctl start arm-rippers.service
```

Enable service at boot:

```bash
sudo systemctl enable arm-rippers.service
```

View service configuration:

```bash
sudo systemctl cat arm-rippers.service
```

## Container Status and Logs

Check running containers:

```bash
sudo docker ps
```

Check ARM container:

```bash
sudo docker ps --filter name=arm-rippers
```

Show last 50 container log lines:

```bash
sudo docker logs arm-rippers --tail 50
```

Follow container logs:

```bash
sudo docker logs -f arm-rippers
```

Inspect container:

```bash
sudo docker inspect arm-rippers
```

Check container health:

```bash
sudo docker inspect arm-rippers \
  --format '{{.State.Health.Status}}'
```

## ARM Startup Script

File:

```text
/home/arm/start_arm_container.sh
```

Current configuration:

```bash
#!/bin/bash

until mount | grep -q '/mnt/jellyfin-freigabe'; do
    echo "/mnt/jellyfin-freigabe is not mounted yet. Waiting..."
    sleep 2
done

echo "Mount is ready, waiting 10 seconds before starting the container..."
sleep 10

docker stop arm-rippers 2>/dev/null || true
docker rm arm-rippers 2>/dev/null || true

docker run -d \
    -p "8080:8080" \
    -e ARM_UID="1001" \
    -e ARM_GID="1001" \
    -v "/home/arm:/home/arm" \
    -v "/home/arm/config:/etc/arm/config" \
    -v "/mnt/jellyfin-freigabe:/mnt/jellyfin-freigabe" \
    --device="/dev/sr0:/dev/sr0" \
    --privileged \
    --restart "always" \
    --name "arm-rippers" \
    --hostname=arm-rippers \
    --cpuset-cpus='2-7' \
    --runtime=runc \
    --workdir=/home/arm \
    automaticrippingmachine/automatic-ripping-machine:latest \
    /sbin/my_init
```

## ARM Configuration

Main configuration file:

```text
/home/arm/config/arm.yaml
```

ARM uses software-based H.265 encoding.

Recommended presets:

```yaml
HB_PRESET_DVD: "H.265 MKV 1080p30"
HB_PRESET_BD: "H.265 MKV 2160p60 4K"
```

Check configured presets:

```bash
sudo grep -nE 'HB_PRESET_DVD|HB_PRESET_BD' \
  /home/arm/config/arm.yaml
```

Edit configuration:

```bash
sudo nano /home/arm/config/arm.yaml
```

Restart ARM after configuration changes:

```bash
sudo systemctl restart arm-rippers.service
```

## Jellyfin Share

ARM writes completed media to:

```text
/mnt/jellyfin-freigabe
```

Check mount:

```bash
findmnt /mnt/jellyfin-freigabe
```

Check contents:

```bash
ls -lah /mnt/jellyfin-freigabe
```

Check SMB server connectivity:

```bash
ping -c 4 192.168.1.10
```

Check SMB port:

```bash
nc -zv 192.168.1.10 445
```

Check CIFS kernel messages:

```bash
sudo dmesg | grep -i cifs | tail -50
```

## Optical Drive

ARM uses:

```text
/dev/sr0
```

Check device:

```bash
ls -l /dev/sr0
```

Check device inside container:

```bash
sudo docker exec arm-rippers ls -l /dev/sr0
```

ARM 2.24.3 includes the corrected mount command:

```text
mount --source /dev/sr0
```

Check implementation:

```bash
sudo docker exec arm-rippers \
  grep -nE 'mount.*(--all|--source|X-mount.mkdir)' \
  /opt/arm/arm/ripper/identify.py
```

Expected result:

```text
arm_subprocess(["mount", "--source", job.devpath])
```

## Backup

Create backup directory:

```bash
sudo mkdir -p ~/backups/$(date +%Y%m%d)
```

Back up ARM configuration:

```bash
sudo cp -a /home/arm/config \
  ~/backups/$(date +%Y%m%d)/config
```

Back up startup script:

```bash
sudo cp -a /home/arm/start_arm_container.sh \
  ~/backups/$(date +%Y%m%d)/
```

Back up service definition:

```bash
sudo cp -a /etc/systemd/system/arm-rippers.service \
  ~/backups/$(date +%Y%m%d)/
```

Back up container inspection:

```bash
sudo docker inspect arm-rippers \
  > ~/backups/$(date +%Y%m%d)/arm-rippers-inspect.json
```

Create compressed ARM configuration backup:

```bash
sudo tar \
  --exclude='/home/arm/media' \
  --exclude='/home/arm/logs' \
  -czf ~/backups/arm-config-$(date +%Y%m%d-%H%M).tar.gz \
  /home/arm
```

## Update Procedure

Pull latest official ARM image:

```bash
sudo docker pull \
  automaticrippingmachine/automatic-ripping-machine:latest
```

Restart ARM service:

```bash
sudo systemctl restart arm-rippers.service
```

Verify container:

```bash
sudo docker ps --filter name=arm-rippers
```

Check ARM version:

```bash
sudo docker exec arm-rippers cat /opt/arm/VERSION
```

Check logs:

```bash
sudo docker logs arm-rippers --tail 50
```

## Docker Image Management

List images:

```bash
sudo docker images
```

Show Docker disk usage:

```bash
sudo docker system df
```

Remove unused images:

```bash
sudo docker image prune
```

Remove a specific obsolete image:

```bash
sudo docker rmi <image-name>
```

## Web Interface

```text
http://<server-ip>:8080
```
