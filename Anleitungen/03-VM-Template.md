# Debian 13 Docker-Template-VM auf Proxmox VE – Installationsanleitung

> Ziel: Eine saubere Debian-13-VM als wiederverwendbares Template für spätere Service-VMs erstellen. Das Template enthält ein schlankes Debian-Grundsystem, SSH, QEMU Guest Agent, Docker Engine und Docker Compose Plugin.

> Voraussetzung: Proxmox VE läuft bereits auf `pve1`, der Storage `local-zfs` ist vorhanden und die Bridge `vmbr0` ist eingerichtet.

---

## 1. Ziel der Template-VM

Diese VM wird nicht produktiv genutzt, sondern dient ausschließlich als Vorlage für spätere VMs.

Geplante Template-VM:

```text
VM-ID: 9000
Name:  debian13-docker-template
OS:    Debian 13 amd64
```

Daraus können später produktive VMs geklont werden, z. B.:

```text
VM 101: service-stack
VM 102: mqtt-zigbee
VM 103: media-tools
VM 104: ai-tools
```

Das Template enthält bewusst nur die gemeinsame Basis:

- Debian 13 Server
- SSH
- QEMU Guest Agent
- Docker Engine
- Docker Compose Plugin
- grundlegende Admin-Tools
- vorbereitete Verzeichnisstruktur unter `/opt`

Nicht ins Template gehören:

- Portainer
- produktive Docker-Stacks
- projektspezifische Konfigurationen
- feste produktive IP-Adressen
- API-Keys oder Secrets

---

## 2. Empfohlene VM-Konfiguration

Für das Template werden moderate Ressourcen verwendet.

```text
VM-ID:       9000
Name:        debian13-docker-template
BIOS:        OVMF / UEFI
Machine:     q35
CPU:         host
vCPU:        2
RAM:         2048 MB
Disk:        32 GB
Storage:     local-zfs
Controller:  VirtIO SCSI single
Disk-Bus:    SCSI
Network:     VirtIO, Bridge vmbr0
QEMU Agent:  aktiviert
Autostart:   nein
```

Diese Werte sind für ein Template ausreichend. Nach dem Klonen können CPU, RAM und Disk je nach Einsatzzweck angepasst werden.

---

## 3. Debian-13-ISO bereitstellen

Das Debian-Netinst-ISO auf den Mac herunterladen oder direkt über die Proxmox-Weboberfläche hochladen.

Empfohlener Ablageort auf Proxmox:

```text
/var/lib/vz/template/iso/
```

### Variante A – Upload über die Proxmox-Weboberfläche

Pfad:

```text
Datacenter → pve1 → local → ISO Images → Upload
```

Dort das Debian-13-AMD64-Netinst-ISO auswählen und hochladen.

### Variante B – Kopieren vom Mac per SCP

Falls die ISO auf dem Desktop des Mac liegt:

```bash
scp ~/Desktop/debian-13.*-amd64-netinst.iso root@192.168.7.40:/var/lib/vz/template/iso/
```

Danach auf Proxmox prüfen:

```bash
ls -lh /var/lib/vz/template/iso/debian-13*amd64-netinst.iso
```

Beispiel:

```text
/var/lib/vz/template/iso/debian-13.0.0-amd64-netinst.iso
```

---

## 4. Template-VM in Proxmox anlegen

In der Proxmox-Shell:

```bash
qm create 9000 \
  --name debian13-docker-template \
  --memory 2048 \
  --cores 2 \
  --cpu host \
  --bios ovmf \
  --machine q35 \
  --ostype l26 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-single \
  --agent enabled=1 \
  --onboot 0
```

EFI-Disk anlegen:

```bash
qm set 9000 --efidisk0 local-zfs:0,efitype=4m,pre-enrolled-keys=0
```

Systemdisk auf `local-zfs` anlegen:

```bash
qm set 9000 --scsi0 local-zfs:32,discard=on,ssd=1
```

ISO einlegen. Der Dateiname muss ggf. angepasst werden:

```bash
qm set 9000 --ide2 local:iso/debian-13.0.0-amd64-netinst.iso,media=cdrom
```

Boot-Reihenfolge setzen:

```bash
qm set 9000 --boot order=ide2\;scsi0
```

Konfiguration prüfen:

```bash
qm config 9000
```

Wichtige Zeilen:

```text
bios: ovmf
machine: q35
memory: 2048
cores: 2
cpu: host
efidisk0: local-zfs:...
scsi0: local-zfs:...,discard=on,ssd=1
ide2: local:iso/debian-13.0.0-amd64-netinst.iso,media=cdrom
net0: virtio=...,bridge=vmbr0
```

---

## 5. Debian-Installer starten

VM starten:

```bash
qm start 9000
```

In der Proxmox-Weboberfläche:

```text
VM 9000 → Console
```

Im Debian-Bootmenü auswählen:

```text
Install
```

Alternativ kann auch der grafische Installer verwendet werden.

---

## 6. Debian-Grundinstallation

Während der Installation folgende Werte setzen.

### Sprache, Region und Tastatur

Empfohlene Werte:

```text
Language: Deutsch oder Englisch
Country:  Luxembourg oder Germany
Locale:   de_DE.UTF-8 oder en_US.UTF-8
Keyboard: German oder passend zur verwendeten Tastatur
```

### Netzwerk

Die Template-VM kann während der Installation per DHCP eine temporäre IP erhalten.

Hostname:

```text
debian13-template
```

Domain:

```text
multiversum.network
```

Wichtig:

Für das Template keine produktive feste IP setzen. Produktive IPs werden später in den geklonten VMs oder per DHCP-Reservierung vergeben.

### Benutzer

Empfehlung:

```text
Root-Login:  Passwort setzen oder Root-Login deaktivieren
User:        marc oder admin
```

Wenn ein normaler Benutzer angelegt wird, sollte dieser später `sudo` nutzen können.

### Partitionierung

Für das Template reicht eine einfache Partitionierung:

```text
Guided - use entire disk
All files in one partition
```

Die ZFS-Ebene liegt bereits auf Proxmox. Innerhalb der VM ist kein ZFS nötig.

### Software-Auswahl

Nur Server-Basis installieren:

```text
[x] SSH server
[x] standard system utilities
[ ] Debian desktop environment
```

Keinen Desktop installieren.

---

## 7. Nach der Installation: ISO entfernen

Nach Abschluss der Debian-Installation startet die VM neu.

Danach die VM herunterfahren oder stoppen:

```bash
qm stop 9000
```

ISO auswerfen:

```bash
qm set 9000 --ide2 none,media=cdrom
```

Boot-Reihenfolge auf die Systemdisk setzen:

```bash
qm set 9000 --boot order=scsi0
```

VM wieder starten:

```bash
qm start 9000
```

---

## 8. Grundsystem aktualisieren

In die Debian-VM einloggen, entweder über Konsole oder per SSH.

System aktualisieren:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
```

Nach dem Neustart erneut einloggen.

---

## 9. Basiswerkzeuge installieren

```bash
sudo apt install -y \
  qemu-guest-agent \
  sudo \
  curl \
  wget \
  git \
  nano \
  vim \
  htop \
  btop \
  ca-certificates \
  gnupg \
  lsb-release \
  unattended-upgrades \
  apt-listchanges
```

QEMU Guest Agent aktivieren:

```bash
sudo systemctl enable --now qemu-guest-agent
```

Auf dem Proxmox-Host prüfen:

```bash
qm agent 9000 ping
```

Wenn keine Fehlermeldung erscheint, funktioniert der Guest Agent.

---

## 10. Benutzer für sudo vorbereiten

Falls der Benutzer noch nicht in der Gruppe `sudo` ist:

```bash
sudo usermod -aG sudo marc
```

Falls der Benutzer anders heißt:

```bash
whoami
```

Dann entsprechend ersetzen.

Nach Gruppenänderungen einmal abmelden oder neu starten:

```bash
sudo reboot
```

---

## 11. Docker Engine installieren

Docker wird über das offizielle Docker-Repository installiert, nicht über das Debian-Paket `docker.io`.

Keyring-Verzeichnis anlegen:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Docker-GPG-Key hinzufügen:

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Rechte setzen:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Docker-Repository hinzufügen:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Paketquellen aktualisieren:

```bash
sudo apt update
```

Docker installieren:

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

Docker aktivieren:

```bash
sudo systemctl enable --now docker
```

Installation testen:

```bash
sudo docker run hello-world
```

Docker Compose testen:

```bash
docker compose version
```

---

## 12. Benutzer zur Docker-Gruppe hinzufügen

Damit Docker später ohne `sudo` genutzt werden kann:

```bash
sudo usermod -aG docker $USER
```

Danach neu anmelden oder neu starten:

```bash
sudo reboot
```

Nach dem Neustart testen:

```bash
docker run hello-world
```

Wenn der Container ohne `sudo` läuft, ist die Docker-Gruppe korrekt aktiv.

---

## 13. Standard-Verzeichnisstruktur vorbereiten

Für spätere Docker-Stacks wird eine klare Struktur unter `/opt` vorbereitet.

```bash
sudo mkdir -p /opt/docker
sudo mkdir -p /opt/stacks
sudo mkdir -p /opt/data
sudo mkdir -p /opt/backups
```

Rechte für den Standardbenutzer setzen, Beispiel `marc`:

```bash
sudo chown -R marc:marc /opt/docker /opt/stacks /opt/data /opt/backups
```

Falls der Benutzer anders heißt:

```bash
whoami
```

Dann den Namen entsprechend anpassen.

Empfohlene spätere Nutzung:

```text
/opt/stacks   → docker-compose.yml pro Dienst oder Stack
/opt/data     → persistente Anwendungsdaten
/opt/backups  → lokale Export-/Zwischenbackups
/opt/docker   → Hilfsdateien, gemeinsame Docker-Ressourcen
```

---

## 14. SSH prüfen

SSH sollte bereits laufen:

```bash
sudo systemctl status ssh
```

Falls nicht:

```bash
sudo systemctl enable --now ssh
```

Die IP der Template-VM anzeigen:

```bash
ip a
```

Vom Mac testen:

```bash
ssh marc@<template-ip>
```

Für das Template wird SSH noch nicht hart eingeschränkt. SSH-Härtung erfolgt später in den produktiven Klonen, sobald SSH-Keys sauber hinterlegt sind.

Mögliche spätere Anpassungen:

```text
PermitRootLogin no
PasswordAuthentication no
```

Wichtig:

Diese Einstellungen erst setzen, wenn der Zugriff per SSH-Key sicher funktioniert.

---

## 15. Aufräumen vor der Template-Erstellung

Paketcache bereinigen:

```bash
sudo apt autoremove -y
sudo apt clean
```

Temporäre Logs können optional reduziert werden:

```bash
sudo journalctl --vacuum-time=1d
```

Shell-History optional leeren:

```bash
history -c
```

Danach die VM herunterfahren:

```bash
sudo shutdown now
```

---

## 16. Snapshot vor Template-Konvertierung

Vor der Konvertierung kann ein Proxmox-Snapshot erstellt werden.

In der Proxmox-Shell:

```bash
qm snapshot 9000 00-clean-debian13-docker-base --description "Clean Debian 13 Docker base before template conversion"
```

Alternativ über die Weboberfläche:

```text
VM 9000 → Snapshots → Take Snapshot
```

Empfehlung:

```text
Name:        00-clean-debian13-docker-base
Include RAM: aus
Description: Clean Debian 13 Docker base before template conversion
```

---

## 17. VM in Template umwandeln

Wichtig:

Die VM muss ausgeschaltet sein.

Dann auf dem Proxmox-Host:

```bash
qm template 9000
```

Die VM erscheint danach in Proxmox als Template.

Name:

```text
debian13-docker-template
```

---

## 18. Produktive VM aus Template klonen

Beispiel: Service-Stack-VM erstellen.

```bash
qm clone 9000 101 --name service-stack --full true --storage local-zfs
```

Ressourcen anpassen:

```bash
qm set 101 --memory 4096 --cores 4
```

Autostart bei produktiven VMs aktivieren:

```bash
qm set 101 --onboot 1
```

VM starten:

```bash
qm start 101
```

---

## 19. Nach dem Klonen: Hostname und Netzwerk anpassen

In der geklonten VM einloggen.

Hostname setzen, Beispiel:

```bash
sudo hostnamectl set-hostname service-stack
```

`/etc/hosts` prüfen:

```bash
sudo nano /etc/hosts
```

Beispiel:

```text
127.0.0.1       localhost
127.0.1.1       service-stack.multiversum.network service-stack
```

Danach rebooten:

```bash
sudo reboot
```

Die feste IP sollte vorzugsweise im Router bzw. DHCP-Server reserviert werden.

Beispiel:

```text
service-stack.multiversum.network → 192.168.1.101
```

---

## 20. Empfohlene VM-Struktur für Jarvis-Umgebung

Geplante Basisstruktur:

```text
VM 100:  jarvis-haos
VM 101:  service-stack
VM 102:  mqtt-zigbee
VM 103:  media-tools
VM 104:  ai-tools
VM 9000: debian13-docker-template
```

Mögliche Aufgabenverteilung:

```text
jarvis-haos     → Home Assistant OS, zentrale Home-Automation
service-stack   → Grocy, Mealie, kleinere HA-nahe Dienste
mqtt-zigbee     → MQTT, Zigbee2MQTT, ggf. USB-Passthrough
media-tools     → Hilfsdienste rund um Medien, Ripping, Konvertierung
ai-tools        → Pepper-/Jarvis-Brücken, APIs, kleine KI-Dienste
```

Jarvis bleibt Home Assistant. Pepper bleibt die Assistant-/KI-Schicht. Die Debian-Docker-VMs bilden die Service-Schicht neben Home Assistant.

---

## Fertig

Damit existiert eine saubere Debian-13-Docker-Basis als Proxmox-Template.

Die Grundlogik lautet:

```text
Einmal sauber installieren
+
Docker und Basiswerkzeuge vorbereiten
+
als Template einfrieren
+
produktive VMs per Full Clone erzeugen
```

Für spätere Dienste wird nicht mehr jedes Mal neu installiert, sondern aus dem Template geklont und anschließend spezifisch angepasst.
