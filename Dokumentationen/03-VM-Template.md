# Debian 13 Docker-Template-VM auf Proxmox VE – Installationsanleitung

> Ziel: Eine saubere Debian-13-VM als wiederverwendbares Proxmox-Template erstellen. Dieses Template dient später als Basis für weitere Service-VMs neben Jarvis/Home Assistant OS.

> Voraussetzung: Proxmox VE läuft bereits auf `pve1`, der Storage `local-zfs` ist vorhanden, die Bridge `vmbr0` ist eingerichtet und das Debian-13-Netinst-ISO wurde auf `local` unter den ISO-Images abgelegt.

---

## 1. Ziel der Template-VM

Die VM wird nicht direkt produktiv genutzt, sondern als Vorlage für spätere VMs eingefroren.

Geplante Template-VM:

```text
VM-ID: 9000
Name:  debian13-docker-template
OS:    Debian GNU/Linux 13 / amd64
```

Das Template enthält bewusst nur die gemeinsame Basis:

- Debian 13 Server ohne Desktop-Umgebung
- SSH-Server
- `sudo`
- QEMU Guest Agent
- Docker Engine
- Docker Compose Plugin
- grundlegende Admin-Werkzeuge
- vorbereitete Verzeichnisstruktur unter `/opt`

Nicht ins Template gehören:

- Portainer
- produktive Docker-Stacks
- projektspezifische Konfigurationen
- feste produktive IP-Adressen
- API-Keys, Tokens oder Secrets

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

Die Debian-Netinst-ISO wird auf Proxmox unter `local` bei den ISO-Images abgelegt.

Pfad in der Proxmox-Weboberfläche:

```text
Datacenter → pve1 → local → ISO Images → Upload
```

Alternativ kann die ISO vom Mac per SCP übertragen werden:

```bash
scp ~/Desktop/debian-13.*-amd64-netinst.iso root@192.168.7.40:/var/lib/vz/template/iso/
```

Auf dem Proxmox-Host prüfen:

```bash
ls -lh /var/lib/vz/template/iso/
```

Beispiel:

```text
debian-13.5.0-amd64-netinst.iso
```

Wichtig:

ISO-Dateien gehören auf `local`, nicht auf `local-zfs`. `local-zfs` wird für VM-Disks und Container-Disks verwendet.

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
qm set 9000 --ide2 local:iso/debian-13.5.0-amd64-netinst.iso,media=cdrom
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
scsihw: virtio-scsi-single
scsi0: local-zfs:...,discard=on,ssd=1,size=32G
ide2: local:iso/debian-13.5.0-amd64-netinst.iso,media=cdrom
net0: virtio=...,bridge=vmbr0
efidisk0: local-zfs:...,efitype=4m,pre-enrolled-keys=0
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

Der normale Textinstaller reicht für diese Server-Template-VM vollständig aus.

---

## 6. Debian-Grundinstallation

Während der Installation folgende Werte setzen.

### Sprache, Region und Tastatur

Empfohlene Werte für dieses Setup:

```text
Language: English
Country:  Luxembourg
Locale:   en_US.UTF-8
Keyboard: French (Switzerland) / Swiss French
Timezone: Europe/Luxembourg
```

Falls Debian bei der Kombination aus Sprache und Land nach einer Fallback-Locale fragt:

```text
United States - en_US.UTF-8
```

### Netzwerk

Hostname:

```text
debian13-template
```

Domain:

```text
multiversum.network
```

Für das Template keine produktive feste IP setzen. Die VM darf während der Installation eine temporäre DHCP-Adresse erhalten.

Beispiel während der Einrichtung:

```text
192.168.7.158
```

Produktive IPs werden später in den geklonten VMs oder per DHCP-Reservierung vergeben.

### Benutzer

Normalen Benutzer anlegen:

```text
Full name: Marc
Username:  marc
```

Root-Passwort kann gesetzt werden. Der Benutzer `marc` erhält später `sudo`-Rechte.

### Partitionierung

Für das Template reicht eine einfache Partitionierung:

```text
Guided - use entire disk
All files in one partition
Finish partitioning and write changes to disk
Write changes to disks: Yes
```

Die ZFS-Ebene liegt bereits auf Proxmox. Innerhalb der VM ist kein ZFS nötig.

### Paketmanager

Zusätzliche Installationsmedien:

```text
Scan extra installation media? No
```

Debian-Mirror:

```text
Country: Germany
Mirror:  deb.debian.org
Proxy:   leer lassen
```

### Software-Auswahl

Wichtig: Nur die Server-Basis installieren.

```text
[ ] Debian desktop environment
[ ] GNOME
[ ] Xfce
[ ] GNOME Flashback
[ ] KDE Plasma
[ ] Cinnamon
[ ] MATE
[ ] LXDE
[ ] LXQt
[ ] web server
[*] SSH server
[*] standard system utilities
[ ] Choose a Debian Blend for installation
```

Keinen Desktop installieren.

### GRUB

Falls der Installer nach dem Bootloader fragt:

```text
Install the GRUB boot loader: Yes
```

Als Ziel die ganze virtuelle Disk auswählen, nicht eine einzelne Partition.

Beispiele:

```text
/dev/sda
```

oder je nach Anzeige:

```text
/dev/vda
```

---

## 7. Nach der Installation: ISO entfernen

Nach Abschluss der Installation die VM neu starten lassen oder stoppen.

Falls die VM gestoppt ist, auf dem Proxmox-Host:

```bash
qm set 9000 --ide2 none,media=cdrom
qm set 9000 --boot order=scsi0
qm start 9000
```

Optional kann der leere CD-ROM-Slot später komplett entfernt werden:

```bash
qm set 9000 --delete ide2
```

Das ist nicht zwingend nötig. Ein leerer CD-ROM-Slot mit `none,media=cdrom` ist harmlos.

---

## 8. Erster SSH-Zugriff

Nach dem Booten die IP in Proxmox prüfen:

```text
VM 9000 → Summary → IPs
```

Beispiel:

```text
192.168.7.158
```

Vom Mac verbinden:

```bash
ssh marc@192.168.7.158
```

Falls der Mac wegen einer früheren Installation einen geänderten Host-Key meldet:

```bash
ssh-keygen -R 192.168.7.158
ssh marc@192.168.7.158
```

---

## 9. `sudo` installieren und Benutzer berechtigen

Minimalinstallationen enthalten `sudo` nicht immer automatisch.

Falls `sudo` fehlt:

```bash
su -
apt update
apt install -y sudo
usermod -aG sudo marc
reboot
```

Nach dem Neustart erneut per SSH verbinden:

```bash
ssh marc@192.168.7.158
```

Dann testen:

```bash
sudo -i
whoami
```

Erwartung:

```text
root
```

---

## 10. Locale prüfen und reparieren

Falls beim Login Warnungen erscheinen wie:

```text
setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory
```

als root ausführen:

```bash
apt update
apt install -y locales
sed -i 's/^# *en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
update-locale LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8
```

Danach prüfen:

```bash
locale
```

Sollzustand:

```text
LANG=en_US.UTF-8
LC_CTYPE=en_US.UTF-8
```

---

## 11. Grundsystem aktualisieren und Basiswerkzeuge installieren

Als root:

```bash
apt full-upgrade -y
apt install -y \
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
  apt-listchanges \
  qemu-guest-agent
```

QEMU Guest Agent aktivieren:

```bash
systemctl enable --now qemu-guest-agent
```

Auf dem Proxmox-Host prüfen:

```bash
qm agent 9000 ping
```

Wenn keine Fehlermeldung erscheint, funktioniert der Guest Agent.

---

## 12. Docker Engine installieren

Docker wird über das Docker-Repository installiert.

Keyring-Verzeichnis anlegen:

```bash
install -m 0755 -d /etc/apt/keyrings
```

Docker-GPG-Key hinzufügen:

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | \
  gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Rechte setzen:

```bash
chmod a+r /etc/apt/keyrings/docker.gpg
```

Docker-Repository hinzufügen:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Paketquellen aktualisieren:

```bash
apt update
```

Docker installieren:

```bash
apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

Docker aktivieren:

```bash
systemctl enable --now docker
```

Installation testen:

```bash
docker version
docker compose version
docker run hello-world
```

Im dokumentierten Stand wurde Docker Compose erfolgreich geprüft:

```text
Docker Compose version v5.1.4
```

---

## 13. Benutzer zur Docker-Gruppe hinzufügen

Damit Docker später ohne `sudo` genutzt werden kann:

```bash
usermod -aG docker marc
```

Danach neu anmelden oder neu starten:

```bash
reboot
```

Nach dem Neustart als `marc` testen:

```bash
docker ps
docker compose version
```

Erwartung:

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
Docker Compose version v5.1.4
```

---

## 14. Standard-Verzeichnisstruktur vorbereiten

Für spätere Docker-Stacks wird eine klare Struktur unter `/opt` vorbereitet.

Als root:

```bash
mkdir -p /opt/docker
mkdir -p /opt/stacks
mkdir -p /opt/data
mkdir -p /opt/backups
chown -R marc:marc /opt/docker /opt/stacks /opt/data /opt/backups
```

Empfohlene spätere Nutzung:

```text
/opt/stacks   → docker-compose.yml pro Dienst oder Stack
/opt/data     → persistente Anwendungsdaten
/opt/backups  → lokale Export-/Zwischenbackups
/opt/docker   → Hilfsdateien, gemeinsame Docker-Ressourcen
```

---

## 15. Dienste und Systemzustand prüfen

Als Benutzer `marc` oder root prüfen:

```bash
hostname
hostname -I
systemctl status qemu-guest-agent --no-pager
systemctl status docker --no-pager
locale
```

Dokumentierter erfolgreicher Zustand:

```text
Hostname:          debian13-template
VM-IP:             192.168.7.158
Docker-Bridge:     172.17.0.1
QEMU Guest Agent:  active (running)
Docker:            active (running)
Locale:            en_US.UTF-8
```

Hinweis:

Wenn `systemctl status` als normaler Benutzer meldet, dass einige Journal-Dateien nicht geöffnet werden konnten, ist das kein Fehler. Das liegt an fehlenden Leserechten für bestimmte Logs. Der Dienststatus selbst ist entscheidend.

---

## 16. Aufräumen vor der Template-Erstellung

In der VM:

```bash
sudo apt autoremove --purge -y
sudo apt clean
```

Optional:

```bash
df -h
```

Danach die VM herunterfahren:

```bash
sudo shutdown now
```

---

## 17. Snapshot vor Template-Konvertierung

Wenn die VM ausgeschaltet ist, auf dem Proxmox-Host einen Snapshot erstellen.

```bash
qm snapshot 9000 00-debian13-docker-template-ready --description "Debian 13 Docker template ready before conversion"
```

Alternativ über die Weboberfläche:

```text
VM 9000 → Snapshots → Take Snapshot
```

Empfehlung:

```text
Name:        00-debian13-docker-template-ready
Include RAM: aus
Description: Debian 13 Docker template ready before conversion
```

---

## 18. VM in Template umwandeln

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

Dokumentierter Zustand:

```text
VM 9000 - Snapshot              OK
VM 9000 - Convert to template   OK
```

---

## 19. Test-Klon erstellen

Zur Kontrolle wird ein Test-Klon erzeugt.

```bash
qm clone 9000 901 --name test-debian13-docker --full true --storage local-zfs
qm start 901
```

IP in Proxmox prüfen:

```text
VM 901 → Summary → IPs
```

Per SSH verbinden:

```bash
ssh marc@<ip-der-test-vm>
```

Prüfen:

```bash
hostname
docker ps
docker compose version
systemctl status qemu-guest-agent --no-pager
```

Wenn alles funktioniert, Test-VM wieder entfernen:

```bash
qm stop 901
qm destroy 901 --purge
```

Dokumentierter Stand:

```text
Test-Klon erfolgreich erstellt
Docker und Docker Compose erfolgreich geprüft
Test-Klon anschließend wieder entfernt
```

---

## 20. Produktive VM aus Template klonen

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

## 21. Nach dem Klonen: Hostname und Netzwerk anpassen

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

## 22. Empfohlene VM-Struktur für Jarvis-Umgebung

Aktuelle Basisstruktur:

```text
VM 100:  jarvis-haos
VM 9000: debian13-docker-template
```

Mögliche nächste produktive VMs:

```text
VM 101:  service-stack
VM 102:  mqtt-zigbee
VM 103:  media-tools
VM 104:  ai-tools
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
Snapshot erstellen
+
als Template einfrieren
+
produktive VMs per Full Clone erzeugen
```

Für spätere Dienste wird nicht mehr jedes Mal neu installiert, sondern aus dem Template geklont und anschließend spezifisch angepasst.
