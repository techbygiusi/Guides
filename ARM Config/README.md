# ARM (Automatic Ripping Machine) – Admin-Cheatsheet
---

## 1) Docker-Basics

```bash
# Container anzeigen
sudo docker ps          # laufend
sudo docker ps -a       # alle (inkl. gestoppt)

# Logs
sudo docker logs -n 200 arm-rippers
sudo docker logs -f arm-rippers

# Shell im Container
sudo docker exec -it arm-rippers bash
```

---

## 2) Container steuern

```bash
# Neustart (liest geänderte Config-Mounts)
sudo docker restart arm-rippers

# Stoppen und Entfernen
sudo docker stop arm-rippers
sudo docker rm arm-rippers
```

**Neu erstellen:**

```bash
sudo docker run -d   --name arm-rippers   --hostname f945c8a15cb5   --cpuset-cpus 2-7   -p 8080:8080   --restart=always   --network bridge   --privileged   --workdir /home/arm   --device /dev/sr0:/dev/sr0   --device /dev/sg0:/dev/sg0   --device /dev/dri:/dev/dri   --group-add video   --group-add render   -e ARM_UID=1001   -e ARM_GID=1001   -e LIBVA_DRIVER_NAME=iHD   -v /home/arm/logs:/home/arm/logs   -v /home/arm/media:/home/arm/media   -v /home/arm/music:/home/arm/music   -v /mnt/jellyfin-freigabe:/mnt/jellyfin-freigabe   -v /home/arm/config:/etc/arm/config   -v /home/arm:/home/arm   arm-rippers:latest /sbin/my_init
```

---

## 3) Images & „Backups“

```bash
# Images auflisten
sudo docker images

# Container als Image sichern (Commit)
sudo docker commit arm-rippers arm-rippers:backup-$(date +%F)

# Image löschen
sudo docker rmi IMAGE_ID
```

---

## 4) Mounts & Rechte

```bash
# Mount prüfen
mount | grep /mnt/jellyfin-freigabe
df -h /mnt/jellyfin-freigabe

# Rechte prüfen/anpassen
ls -ld /mnt/jellyfin-freigabe
sudo chown -R 1001:1001 /home/arm
sudo chmod -R 0777 /mnt/jellyfin-freigabe
```

---

## 5) ARM-Konfiguration

```bash
# Config bearbeiten (Host)
sudo nano /home/arm/config/arm.yaml

# Container neu laden
sudo docker restart arm-rippers
```

Beispiel `hb_args_bd` (VAAPI-HEVC, BD):

```yaml
hb_args_bd: >
  --encoder vaapi_h265 --quality=20
  --subtitle scan --subtitle-forced
  --all-audio
  --audio-copy-mask dtshd,dts,ac3,eac3,truehd
  --audio-fallback av_aac
  --crop 138:138:0:0
```

---

## 6) VA-API / QSV prüfen & Kurztests

```bash
# Host – VA-API Treiber prüfen
vainfo | head -n 20

# Container – Device-Nodes vorhanden?
sudo docker exec -it arm-rippers bash -lc 'ls -l /dev/dri'

# 10-Sekunden-Test mit VAAPI-HEVC (im Container)
sudo docker exec -it arm-rippers bash -lc '
HandBrakeCLI -i /home/arm/media/raw/Asteroid-City/"Asteroid City"_t00.mkv  -o /home/arm/logs/vaapi-test.mkv --encoder vaapi_h265 --quality 22  --stop-at duration:10'
```

---

## 7) Datenbank (SQLite) Rollback/Restore

```bash
# Container stoppen
sudo docker stop arm-rippers

cd /home/arm/db
# aktuelle DB sichern
cp -a arm.db arm.db_pre-restore_$(date +%F_%H%M)

# WAL/SHM löschen & Snapshot aktivieren
rm -f arm.db-wal arm.db-shm
cp -a arm.db_migration_2025-08-30_1408 arm.db
sudo chown 1001:1001 arm.db
chmod 660 arm.db

# ARM wieder starten
sudo docker start arm-rippers

# Logs checken
sudo docker logs -n 200 arm-rippers | grep -E "Alembic|Database version"
```

---

## 8) Typische ARM-Pfade

```
/home/arm/config/arm.yaml            # ARM-Config
/home/arm/logs/                      # ARM-Logs
/home/arm/media/raw/                 # MakeMKV-Remuxe
/home/arm/media/transcode/           # HandBrake-Ausgaben
/mnt/jellyfin-freigabe/              # SMB-Ziel für Jellyfin
```

```
/mnt/jellyfin-freigabe/movies
```

```
/mnt/jellyfin-freigabe/tv
```

```
/mnt/jellyfin-freigabe/unidentified
```
---

## 9) Systemservice / Docker-Daemon

```bash
# Docker-Dienst
sudo systemctl status docker
sudo systemctl restart docker

# Reboot klassisch
sudo reboot
```

---

## 10) Neustartservice (Einhängen des externen Mounts im Container)

```
sudo nano /etc/systemd/system/restart_docker_container.service
```

```
Description=Restart Docker Container After Boot
After=network.target docker.service

[Service]
ExecStart=/usr/local/bin/restart_docker_container.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```
sudo nano /usr/local/bin/restart_docker_container.sh
```

```
#!/bin/bash

# Warte 20 Sekunden nach Boot
sleep 20

# Stoppe und Starte den Conatainer
sudo docker stop 4465c2bd6461
sudo docker start 4465c2bd6461
```

