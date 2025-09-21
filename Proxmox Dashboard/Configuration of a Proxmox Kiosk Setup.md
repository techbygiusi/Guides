# 🖥️ Proxmox Kiosk Mode Setup

This guide explains how to set up a Proxmox/Debian host to boot directly into a Chromium-based dashboard in **kiosk mode**, without showing the Linux console.

---

## 1. Install minimal GUI stack
On the Proxmox/Debian host:
```bash
apt update
apt install --no-install-recommends xserver-xorg x11-xserver-utils xinit openbox chromium unclutter
```

---

## 2. Create kiosk user
```bash
adduser kiosk
```

---

## 3. Enable auto-login on TTY1
Create a systemd override for `getty@tty1.service`:
```bash
systemctl edit getty@tty1
```

Insert the following:
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin kiosk --noclear %I $TERM
```

Reload systemd:
```bash
systemctl daemon-reexec
```

---

## 4. Auto-start X and Chromium
Edit `/home/kiosk/.bash_profile`:
```bash
if [[ -z $DISPLAY ]] && [[ $(tty) == /dev/tty1 ]]; then
    exec startx
fi
```

---

## 5. X session config
Create `/home/kiosk/.xinitrc`:
```bash
#!/bin/bash
openbox-session &

# Disable DPMS / screen blanking
xset -dpms
xset s off
xset s noblank

# Hide mouse cursor
unclutter -display :0 -idle 0 &

# Start Chromium in kiosk mode
chromium \
  --kiosk \
  --noerrdialogs \
  --disable-features=Translate,TranslateUI,TranslateSubFrames \
  --disable-translate \
  --user-data-dir=/tmp/chrome \
  https://[your-ip-address-or-domain]
```

Make it executable:
```bash
chmod +x /home/kiosk/.xinitrc
```

---

## 6. Apply changes and reboot
```bash
systemctl daemon-reexec
reboot
```

---

✅ After reboot, the system will auto-login as the `kiosk` user, start the X server, and launch Chromium in fullscreen kiosk mode, pointing to your dashboard.
