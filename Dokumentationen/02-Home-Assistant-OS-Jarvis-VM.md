# Home Assistant OS 17.3 auf Proxmox VE – Jarvis-VM Installationsanleitung

> Ziel: Home Assistant OS 17.3 als virtuelle Maschine auf Proxmox VE 9.2.x installieren. Diese VM bildet künftig **Jarvis**, also die zentrale Home-Automation-Instanz im Multiversum-Netzwerk.

> Voraussetzung: Proxmox VE läuft bereits auf dem Intel NUC. Der Hostname bleibt `pve1`, der spätere FQDN lautet `pve1.multiversum.network`. Updates wurden bereits durchgeführt. Als Storage sind `local` und `local-zfs` vorhanden. ZFS wurde bewusst wegen Snapshots gewählt.

---

## 1. Ausgangslage prüfen

Auf dem Proxmox-Host in der Shell prüfen:

```bash
pveversion
ip a
ip r
cat /etc/network/interfaces
cat /etc/hosts
```

Aktueller Stand während der Einrichtung:

```text
Proxmox VE: 9.2.3
Kernel:     7.0.6-2-pve
Host:       pve1
CPU:        Intel i7-10710U, 12 Threads
RAM:        16 GB
Storage:    local, local-zfs
```

Aktuelles Arbeitsnetz während der Einrichtung:

```text
pve1:    192.168.7.40/24
Gateway: 192.168.7.254
Bridge:  vmbr0
NIC:     nic0
```

Die Bridge `vmbr0` muss aktiv sein und auf die physische Netzwerkkarte zeigen:

```text
iface nic0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.7.40/24
        gateway 192.168.7.254
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
```

### Wichtiger Hinweis

Die Installation wurde zunächst im Arbeitsnetz durchgeführt. Die finale Netzwerkkonfiguration erfolgt später im Heimnetz.

Solange der NUC noch im Arbeitsnetz hängt, sollte die Proxmox-IP nicht vorschnell auf die Heimnetz-IP geändert werden, da sonst der Zugriff auf die Weboberfläche verloren gehen kann.

---

## 2. Home Assistant OS Image vorbereiten

Für Proxmox wird das Home-Assistant-OS-Image im Format `qcow2` verwendet.

Verwendete Version:

```text
Home Assistant OS: 17.3
Image:             haos_ova-17.3.qcow2
```

Falls das Image auf dem Mac liegt, z. B. auf dem Desktop, wird es per `scp` auf den Proxmox-Host kopiert.

Auf dem Mac ausführen:

```bash
scp ~/Desktop/haos_ova-17.3.qcow2 root@192.168.7.40:/root/
```

Falls beim ersten Verbindungsaufbau eine Host-Key-Abfrage erscheint:

```text
Are you sure you want to continue connecting?
```

mit `yes` bestätigen.

Danach auf dem Proxmox-Host prüfen:

```bash
ls -lh /root/haos_ova-17.3.qcow2
```

---

## 3. VM-Grundkonfiguration festlegen

Jarvis wird als eigene HAOS-VM angelegt.

Empfohlene Konfiguration für den Intel NUC:

```text
VM-ID:       100
Name:        jarvis-haos
BIOS:        OVMF / UEFI
Machine:     q35
CPU:         host
vCPU:        4 Cores
RAM:         4096 MB
Ballooning:  deaktiviert
Disk:        64 GB auf local-zfs
Controller:  VirtIO SCSI single
Disk-Bus:    SCSI
Network:     VirtIO, Bridge vmbr0
Autostart:   aktiviert
```

### Warum diese Werte?

Home Assistant OS selbst läuft auch mit weniger Ressourcen. Für Jarvis wird jedoch bewusst etwas Reserve eingeplant, da später weitere Integrationen, Dashboards, Add-ons, Energieverwaltung, Sprachfunktionen oder lokale Schnittstellen hinzukommen können.

Die VM bleibt trotzdem schlank genug, damit auf dem NUC weitere VMs oder LXCs betrieben werden können.

---

## 4. VM-Hülle erstellen

In der Proxmox-Shell ausführen:

```bash
qm create 100 \
  --name jarvis-haos \
  --memory 4096 \
  --cores 4 \
  --cpu host \
  --bios ovmf \
  --machine q35 \
  --ostype l26 \
  --net0 virtio,bridge=vmbr0 \
  --scsihw virtio-scsi-single \
  --agent enabled=1 \
  --onboot 1
```

Danach die permanente EFI-Disk anlegen:

```bash
qm set 100 --efidisk0 local-zfs:0,efitype=4m,pre-enrolled-keys=0
```

### Wichtiger Hinweis zur EFI-Disk

Bei Verwendung von `OVMF / UEFI` sollte eine permanente EFI-Disk vorhanden sein.

Falls beim Start der VM folgende Warnung erscheint:

```text
WARN: no efidisk configured! Using temporary efivars disk.
```

wurde die EFI-Disk noch nicht angelegt oder nicht korrekt übernommen.

Dann die VM stoppen:

```bash
qm stop 100
```

EFI-Disk nachträglich anlegen:

```bash
qm set 100 --efidisk0 local-zfs:0,efitype=4m,pre-enrolled-keys=0
```

Danach die Konfiguration prüfen:

```bash
qm config 100
```

Es sollte eine Zeile `efidisk0` vorhanden sein.

---

## 5. HAOS-Image in Proxmox importieren

Das zuvor kopierte `qcow2`-Image wird nach `local-zfs` importiert:

```bash
qm importdisk 100 /root/haos_ova-17.3.qcow2 local-zfs
```

Danach die VM-Konfiguration prüfen:

```bash
qm config 100
```

Nach dem Import erscheint die Disk zunächst als nicht verwendete Disk, z. B.:

```text
unused0: local-zfs:vm-100-disk-0
```

---

## 6. Importierte Disk einhängen

Die importierte Disk wird als SCSI-Disk eingebunden.

Wenn `qm config 100` diese Zeile zeigt:

```text
unused0: local-zfs:vm-100-disk-0
```

dann ausführen:

```bash
qm set 100 --scsi0 local-zfs:vm-100-disk-0,discard=on,ssd=1
```

Falls der Disk-Name abweicht, muss der tatsächliche Name aus der `unused0`-Zeile verwendet werden.

Danach die Boot-Reihenfolge setzen:

```bash
qm set 100 --boot order=scsi0
```

Disk auf 64 GB erweitern:

```bash
qm resize 100 scsi0 64G
```

Ballooning deaktivieren:

```bash
qm set 100 --balloon 0
```

---

## 7. VM-Konfiguration prüfen

Vor dem ersten Start prüfen:

```bash
qm config 100
```

Die Konfiguration sollte sinngemäß folgende Zeilen enthalten:

```text
agent: enabled=1
bios: ovmf
boot: order=scsi0
cores: 4
cpu: host
efidisk0: local-zfs:vm-100-disk-...,efitype=4m,pre-enrolled-keys=0
machine: q35
memory: 4096
name: jarvis-haos
net0: virtio=...,bridge=vmbr0
onboot: 1
ostype: l26
scsi0: local-zfs:vm-100-disk-0,discard=on,ssd=1
scsihw: virtio-scsi-single
```

Wichtig sind vor allem:

```text
bios: ovmf
boot: order=scsi0
efidisk0: local-zfs:...
scsi0: local-zfs:vm-100-disk-...
net0: virtio=...,bridge=vmbr0
```

---

## 8. VM starten

Die VM starten:

```bash
qm start 100
```

Danach die Konsole öffnen:

```text
VM 100 → Console
```

Oder in der Proxmox-Weboberfläche:

```text
pve1 → 100 (jarvis-haos) → Console
```

Nach erfolgreichem Boot zeigt Home Assistant OS die Kommandozeile und Systeminformationen an.

Erwartete Ausgabe:

```text
Home Assistant OS Version: 17.3
Home Assistant Supervisor is running!
System is ready! Use browser or app to configure.
```

Während der Einrichtung wurde folgende IP per DHCP vergeben:

```text
IPv4: 192.168.7.154/24
```

Der erste Zugriff erfolgt über:

```text
http://192.168.7.154:8123
```

Alternativ, falls mDNS funktioniert:

```text
http://homeassistant.local:8123
```

---

## 9. Ersten Proxmox-Snapshot erstellen

Sobald Home Assistant OS sauber gebootet hat und die Onboarding-Seite erreichbar ist, wird vor weiteren Änderungen ein erster Snapshot erstellt.

In der Proxmox-Weboberfläche:

```text
VM 100 → Snapshots → Take Snapshot
```

Empfohlene Werte:

```text
Name:        00-clean-haos-17-3-first-boot
Include RAM: aus
Description: clean install, no onboarding yet
```

Alternativ per Shell:

```bash
qm snapshot 100 00-clean-haos-17-3-first-boot --description "clean install, no onboarding yet"
```

### Warum ohne RAM?

Für einen sauberen Installationsstand ist der Festplattenzustand relevant. Der aktuelle RAM-Zustand wird nicht benötigt.

Snapshots sind praktisch für schnelle Rücksprünge, ersetzen aber auch hier kein externes Backup.

---

## 10. Home Assistant Onboarding starten

Im Browser öffnen:

```text
http://192.168.7.154:8123
```

Dann:

```text
Create my smart home
```

Empfohlene Grundeinstellungen:

```text
Instanzname: Jarvis
Benutzer:    Marc
Sprache:     Deutsch oder Englisch
Zeitzone:    Europe/Luxembourg
Einheiten:   Metrisch, Celsius, EUR
```

### Wichtig im Arbeitsnetz

Da die Installation zunächst im Arbeitsnetz durchgeführt wird, sollten automatisch erkannte Geräte aus diesem Netz nicht produktiv übernommen werden.

Empfehlung:

```text
Discovered Devices zunächst überspringen oder nicht dauerhaft konfigurieren.
```

Die eigentliche Geräteintegration erfolgt später im Heimnetz.

---

## 11. Zweiten Snapshot nach Onboarding erstellen

Nach abgeschlossenem Onboarding und bevor Integrationen installiert werden, einen zweiten Snapshot erstellen.

Empfohlene Werte:

```text
Name:        01-jarvis-onboarded-basic
Include RAM: aus
Description: Home Assistant onboarding completed, no integrations configured yet
```

Alternativ per Shell:

```bash
qm snapshot 100 01-jarvis-onboarded-basic --description "Home Assistant onboarding completed, no integrations configured yet"
```

Damit gibt es zwei saubere Rücksetzpunkte:

```text
00-clean-haos-17-3-first-boot
01-jarvis-onboarded-basic
```

---

## 12. Finale IP und DNS im Heimnetz vorbereiten

Die aktuelle IP `192.168.7.154` stammt aus dem Arbeitsnetz und ist nicht final.

Im Heimnetz sollte Jarvis per DHCP-Reservierung eine feste IP erhalten.

Beispiel:

```text
Name:   jarvis
FQDN:   jarvis.multiversum.network
IP:     192.168.1.50/24
DNS:    Router oder lokaler DNS
```

Empfehlung:

- feste DHCP-Reservierung im Router oder DHCP-Server
- DNS-Eintrag für `jarvis.multiversum.network`
- keine unnötige statische Netzwerkkonfiguration innerhalb von HAOS

Späterer Zugriff:

```text
http://jarvis.multiversum.network:8123
```

oder:

```text
http://192.168.1.50:8123
```

---

## 13. Netzwerkumzug nach Hause

Vor dem Umzug sollte Proxmox noch nicht produktiv auf das Heimnetz umgestellt werden, solange Zugriff im Arbeitsnetz benötigt wird.

Wenn der NUC nach Hause umzieht:

1. VM sauber herunterfahren
2. Proxmox-Host sauber herunterfahren
3. NUC im Heimnetz per LAN anschließen
4. Proxmox starten
5. Proxmox-IP auf die Heimnetz-IP ändern, falls noch nicht geschehen
6. Router/DHCP/DNS für `pve1` und `jarvis` einrichten

Für `pve1` später z. B.:

```text
pve1.multiversum.network → 192.168.1.40
```

Für Jarvis später z. B.:

```text
jarvis.multiversum.network → 192.168.1.50
```

---

## 14. Erste Home-Assistant-Grundeinstellungen

Nach dem Onboarding in Home Assistant prüfen:

```text
Settings → System → General
```

Empfohlen:

```text
Name:      Jarvis
Location:  Heimadresse setzen
Unit:      Metric
Currency:  EUR
Timezone:  Europe/Luxembourg
```

Danach Updates prüfen:

```text
Settings → System → Updates
```

Benutzerprofil öffnen und den erweiterten Modus aktivieren:

```text
Advanced Mode: on
```

Noch nicht installieren:

```text
HACS
Zigbee
Z-Wave
Bluetooth
OpenWB
Grocy
Mealie
Frigate
Voice Assistant
LLM/KI-Integration
```

Zuerst muss die HAOS-Basis stabil stehen.

---

## 15. Home Assistant Backup erstellen

Zusätzlich zu Proxmox-Snapshots sollte Home Assistant ein internes Backup erhalten.

In Home Assistant:

```text
Settings → System → Backups → Create backup
```

Empfohlener Name:

```text
01-jarvis-basic-onboarding
```

### Einordnung

Home-Assistant-Backups sichern die HA-Konfiguration und Add-ons aus Sicht von Home Assistant.

Proxmox-Snapshots sichern den VM-Zustand auf Hypervisor-Ebene.

Beides ist sinnvoll, ersetzt aber kein externes Backup auf ein separates Ziel.

---

## 16. Externes Backup-Ziel vorbereiten

Das Backup-Konzept ist noch offen. Für den Start wird ein Synology-NFS-Share als Proxmox-Backupziel bevorzugt.

Empfohlene Struktur:

```text
Synology Share: proxmox-backup
Protokoll:      NFS
Proxmox ID:     synology-backup
Content:        VZDump backup file
```

In Proxmox hinzufügen über:

```text
Datacenter → Storage → Add → NFS
```

Danach einen Backup-Job für Jarvis anlegen:

```text
Datacenter → Backup → Add
```

Empfohlener Startwert:

```text
VM:          100 jarvis-haos
Schedule:    täglich nachts
Mode:        Snapshot
Compression: zstd
Retention:   7 daily, 4 weekly, 3 monthly
```

Ein Proxmox Backup Server auf der Synology wurde diskutiert, wird aber vorerst nicht als erste Produktivlösung gewählt.

---

## 17. USB-Passthrough für spätere Hardware

Für Zigbee-, Z-Wave- oder Bluetooth-Hardware wird später USB-Passthrough benötigt.

Zunächst auf dem Proxmox-Host prüfen:

```bash
lsusb
```

Beispiel:

```text
Bus 003 Device 003: ID 1a86:55d4 Zigbee USB Dongle
```

In Proxmox:

```text
VM 100 → Hardware → Add → USB Device
```

Empfehlung:

```text
Use USB Vendor/Device ID
```

Nicht bevorzugt:

```text
Use USB Port
```

### Warum Vendor/Device ID?

Die Durchreichung über Vendor/Device ID ist meist stabiler, wenn ein USB-Gerät einmal abgezogen oder an einen anderen Port gesteckt wird.

Nach dem Hinzufügen eines USB-Geräts:

```text
VM sauber herunterfahren
USB-Gerät hinzufügen
VM wieder starten
```

Erst danach die entsprechende Integration in Home Assistant einrichten.

---

## Fertig

Damit läuft Home Assistant OS 17.3 als VM auf Proxmox VE 9.2.x.

Die Rolle ist klar getrennt:

```text
Jarvis = Home Assistant / zentrale Home-Automation
Pepper = Assistant- und KI-Schicht
```

Jarvis soll nicht stumpf sein, aber auch kein Smalltalk-Assistent werden. Die Basis ist daher bewusst stabil, sauber und appliance-nah aufgebaut.

Die empfohlene Grundlogik lautet:

```text
HAOS als stabile VM
+
ZFS-Snapshots für schnelle Rücksprünge
+
Home-Assistant-Backups für Konfiguration
+
externe Proxmox-Backups auf Synology
+
später optionale Hardware-Integrationen per USB-Passthrough
```
