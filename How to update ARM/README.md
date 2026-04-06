# ARM Ripper – Docker Cheatsheet
---

## 📁 Verzeichnisstruktur

```
/home/giuseppe/
├── run-arm-rippers.sh         # Start-Script (aktiv)
├── arm-rippers-config.json    # Konfiguration (aktiv)
├── docker-setup.sh            # Original Setup-Script (ARM-Projekt)
└── backups/
    └── YYYYMMDD/              # Archiv pro Update-Datum
        ├── run-arm-rippers.sh.bak
        ├── arm-rippers-config.json.bak-YYYYMMDD
        ├── arm-rippers-inspect-YYYYMMDD.json
        └── docker-setup.sh.bak

/home/arm/
├── config/                    # ARM-Konfiguration (Volume)
├── logs/                      # Logs (Volume)
├── media/                     # Medien (Volume)
└── music/                     # Musik (Volume)

/mnt/jellyfin-freigabe/        # Jellyfin-Freigabe (Volume)
```

---

## 🔍 Status & Logs

```bash
# Container-Status prüfen
sudo docker ps

# Letzte 50 Log-Zeilen
sudo docker logs arm-rippers --tail 50

# Logs live verfolgen
sudo docker logs arm-rippers -f

# Container Details
sudo docker inspect arm-rippers
```

---

## ▶️ Container starten / stoppen

```bash
# Starten (via Script)
sudo bash run-arm-rippers.sh

# Stoppen
sudo docker stop arm-rippers

# Stoppen + entfernen
sudo docker stop arm-rippers && sudo docker rm arm-rippers

# Neu starten (restart)
sudo docker restart arm-rippers
```

---

## 🔄 Update-Prozedur

### 1. Backup anlegen

```bash
# Altes Image sichern (vor dem Überschreiben!)
sudo docker tag arm-rippers:backup-2025-08-30 arm-rippers:pre-update-$(date +%Y%m%d)

# Scripts & Config sichern
sudo cp run-arm-rippers.sh run-arm-rippers.sh.bak
sudo cp arm-rippers-config.json arm-rippers-config.json.bak-$(date +%Y%m%d)
sudo docker inspect arm-rippers > arm-rippers-inspect-$(date +%Y%m%d).json
sudo cp -r /home/arm/config /home/arm/config.bak-$(date +%Y%m%d)
```

### 2. Neues Image pullen

```bash
sudo docker pull automaticrippingmachine/automatic-ripping-machine:latest
```

### 3. Neu taggen (Run-Script bleibt unverändert)

```bash
sudo docker tag automaticrippingmachine/automatic-ripping-machine:latest arm-rippers:backup-2025-08-30
```

### 4. Container neu starten

```bash
sudo docker stop arm-rippers
sudo docker rm arm-rippers
sudo bash run-arm-rippers.sh
```

### 5. Prüfen

```bash
sudo docker ps
sudo docker logs arm-rippers --tail 50
```

---

## ⏪ Rollback

```bash
sudo docker stop arm-rippers && sudo docker rm arm-rippers
sudo docker tag arm-rippers:pre-update-YYYYMMDD arm-rippers:backup-2025-08-30
sudo bash run-arm-rippers.sh
```

> **Datum anpassen:** `YYYYMMDD` → z.B. `20260406`

---

## 🗂️ Backup-Archiv aufräumen

```bash
# Backup-Ordner mit heutigem Datum erstellen
sudo mkdir -p ~/backups/$(date +%Y%m%d)

# Alle .bak und Inspect-Dateien verschieben
sudo mv ~/arm-rippers-config.json.bak-* ~/backups/$(date +%Y%m%d)/
sudo mv ~/arm-rippers-inspect-*.json ~/backups/$(date +%Y%m%d)/
sudo mv ~/docker-setup.sh.bak ~/backups/$(date +%Y%m%d)/
sudo mv ~/run-arm-rippers.sh.bak ~/backups/$(date +%Y%m%d)/

# Prüfen
ls ~/backups/$(date +%Y%m%d)/
```

---

## 🐳 Docker Image Verwaltung

```bash
# Alle lokalen Images anzeigen
sudo docker images

# Ungenutzte Images aufräumen
sudo docker image prune

# Alle ungenutzten Ressourcen aufräumen (Images, Container, Volumes, Netzwerke)
sudo docker system prune

# Speicherverbrauch anzeigen
sudo docker system df

# Spezifisches Image löschen
sudo docker rmi arm-rippers:pre-update-YYYYMMDD
```

---

## ⚙️ Container-Konfiguration (Referenz)

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

---

## 🌐 Web UI

```
http://<server-ip>:8080
```

---

## ⚠️ Bekannte Hinweise

| Meldung | Bedeutung | Aktion |
|---|---|---|
| `Database is not current, update required` | DB-Schema veraltet nach Update | Automatisch beim nächsten Job, oder manuell triggern |
| `Intel QuickSync supported!` | Hardware-Encoding aktiv | ✅ OK |
| `Handbrake call successful` | Handbrake funktioniert | ✅ OK |

