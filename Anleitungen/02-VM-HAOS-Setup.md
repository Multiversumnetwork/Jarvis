# Home Assistant OS als Jarvis-VM auf Proxmox VE – Installationsanleitung

> Ziel: Home Assistant OS als saubere VM auf Proxmox VE installieren. Diese VM bildet künftig **Jarvis**, also die zentrale Home-Automation-Instanz im Multiversum-Netzwerk.

> Voraussetzung: Proxmox VE 9.2.x läuft bereits auf dem Intel NUC. Der Host heißt `pve1`, der spätere FQDN lautet `pve1.multiversum.network`. Storage `local` und `local-zfs` sind vorhanden. ZFS wurde bewusst wegen Snapshots gewählt.

---

## 1. Ausgangslage prüfen

Vor dem Anlegen der VM sollte geprüft werden, ob Proxmox sauber läuft und ob die Netzwerk-Bridge vorhanden ist.

In der Proxmox-Shell:

```bash
ip a
ip r
cat /etc/network/interfaces
cat /etc/hosts
```

Erwartet wird eine aktive Bridge `vmbr0`, die an der physischen Netzwerkkarte des NUC hängt.

Beispiel aus der Erstinstallation:

```text
vmbr0: 192.168.7.40/24
Gateway: 192.168.7.254
Bridge-Port: nic0
```

Wichtig:

```text
Die hier gezeigte IP stammt aus dem Arbeitsnetz.
Die finale Heimnetz-IP wird später zu Hause gesetzt.
```

Die VM kann trotzdem bereits vorbereitet und installiert werden, solange sie über `vmbr0` ins aktuelle Netzwerk kommt.

---

## 2. HAOS-Image herunterladen

Für Home Assistant OS wird das offizielle OVA-/qcow2-Image verwendet.

Auf dem Mac oder direkt auf Proxmox herunterladen:

```text
haos_ova-17.3.qcow2.xz
```

Falls die Datei noch komprimiert ist, muss sie zuerst entpackt werden:

```bash
xz -d -v haos_ova-17.3.qcow2.xz
```

Danach sollte die Datei vorliegen als:

```text
haos_ova-17.3.qcow2
```

Prüfen:

```bash
ls -lh haos_ova-17.3.qcow2
```

---

## 3. Image vom Mac auf Proxmox kopieren

Falls das HAOS-Image lokal auf dem Mac liegt, wird es per `scp` auf den Proxmox-Host kopiert.

Auf dem Mac-Terminal ausführen:

```bash
scp ~/Desktop/haos_ova-17.3.qcow2 root@192.168.7.40:/root/
```

Dabei ist `192.168.7.40` die aktuelle IP von `pve1` im Arbeitsnetz.

Falls der SSH-Host noch unbekannt ist, die Nachfrage bestätigen:

```text
Are you sure you want to continue connecting?
yes
```

Danach in der Proxmox-Shell prüfen:

```bash
ls -lh /root/haos_ova-17.3.qcow2
```

---

## 4. VM-Grundkonfiguration festlegen

Für Jarvis wird folgende VM verwendet:

```text
VM-ID:        100
Name:         jarvis-haos
Rolle:        Home Assistant OS / Jarvis
BIOS:         OVMF / UEFI
Machine:      q35
CPU:          host
Cores:        4
RAM:          4096 MB
Disk:         64 GB auf local-zfs
SCSI:         VirtIO SCSI single
Netzwerk:     VirtIO, Bridge vmbr0
Autostart:    aktiviert
```

Warum diese Werte?

- 4 vCPU geben Home Assistant genug Luft für Integrationen, Add-ons und spätere Erweiterungen.
- 4 GB RAM sind ein sinnvoller Startwert für eine zentrale HAOS-Instanz.
- 64 GB Disk verhindern, dass Add-ons, Backups, Recorder-Datenbank und Logs zu früh knapp werden.
- `local-zfs` erlaubt schnelle lokale Snapshots.
- OVMF/UEFI und q35 sind eine moderne, saubere VM-Basis.

---

## 5. VM anlegen

In der Proxmox-Shell:

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

Danach eine permanente EFI-Disk anlegen:

```bash
qm set 100 --efidisk0 local-zfs:0,efitype=4m,pre-enrolled-keys=0
```

Wichtig:

Ohne `efidisk0` startet die VM zwar unter Umständen, Proxmox meldet dann aber:

```text
WARN: no efidisk configured! Using temporary efivars disk.
```

Für eine produktive HAOS-VM sollte daher immer eine permanente EFI-Disk vorhanden sein.

---

## 6. HAOS-Disk importieren

Das HAOS-qcow2-Image wird jetzt nach `local-zfs` importiert:

```bash
qm importdisk 100 /root/haos_ova-17.3.qcow2 local-zfs
```

Danach die VM-Konfiguration prüfen:

```bash
qm config 100
```

Nach dem Import erscheint die Disk zunächst als `unused0`, zum Beispiel:

```text
unused0: local-zfs:vm-100-disk-0
```

---

## 7. Importierte Disk als Boot-Disk einhängen

Die importierte Disk wird als `scsi0` eingehängt.

Wenn in der Konfiguration steht:

```text
unused0: local-zfs:vm-100-disk-0
```

dann:

```bash
qm set 100 --scsi0 local-zfs:vm-100-disk-0,discard=on,ssd=1
```

Falls die Disk anders heißt, z. B. `vm-100-disk-1`, entsprechend anpassen:

```bash
qm set 100 --scsi0 local-zfs:vm-100-disk-1,discard=on,ssd=1
```

Danach die Boot-Reihenfolge setzen:

```bash
qm set 100 --boot order=scsi0
```

---

## 8. Disk auf 64 GB erweitern

Die importierte HAOS-Disk wird auf 64 GB erweitert:

```bash
qm resize 100 scsi0 64G
```

Home Assistant OS sollte die Erweiterung beim Start bzw. im Betrieb übernehmen.

Zusätzlich Ballooning deaktivieren:

```bash
qm set 100 --balloon 0
```

Für Jarvis als zentrale Home-Automation-Instanz ist fester RAM sinnvoller als dynamisches Ballooning.

---

## 9. VM-Konfiguration prüfen

Vor dem ersten Start:

```bash
qm config 100
```

Die wichtigsten Zeilen sollten ungefähr so aussehen:

```text
agent: enabled=1
bios: ovmf
boot: order=scsi0
cores: 4
cpu: host
efidisk0: local-zfs:vm-100-disk-...
machine: q35
memory: 4096
name: jarvis-haos
net0: virtio=...,bridge=vmbr0
onboot: 1
ostype: l26
scsi0: local-zfs:vm-100-disk-...,discard=on,ssd=1
scsihw: virtio-scsi-single
```

Wichtig:

```text
boot: order=scsi0
efidisk0: vorhanden
scsi0: vorhanden
net0: bridge=vmbr0
```

---

## 10. VM starten

Die VM starten:

```bash
qm start 100
```

Danach in der Proxmox-Weboberfläche öffnen:

```text
VM 100 → Console
```

Bei erfolgreichem Start erscheint die Home-Assistant-CLI.

Beispiel:

```text
Home Assistant OS Version: 17.3
IPv4 address: 192.168.7.154/24
Home Assistant URL: http://homeassistant.local:8123
System is ready!
```

---

## 11. Erster Zugriff auf Home Assistant

Im Browser aufrufen:

```text
http://<jarvis-ip>:8123
```

Beispiel aus dem Arbeitsnetz:

```text
http://192.168.7.154:8123
```

Alternativ kann mDNS funktionieren:

```text
http://homeassistant.local:8123
```

Falls `homeassistant.local` nicht auflöst, direkt die IP verwenden.

---

## 12. Ersten Proxmox-Snapshot erstellen

Bevor das Onboarding gestartet wird, einen sauberen Snapshot anlegen.

Empfohlener Name:

```text
00-clean-haos-17-3-first-boot
```

Beschreibung:

```text
clean install, no onboarding yet
```

In der Proxmox-Weboberfläche:

```text
VM 100 → Snapshots → Take Snapshot
```

Wichtig:

```text
Include RAM: deaktivieren
```

Alternativ per Shell:

```bash
qm snapshot 100 00-clean-haos-17-3-first-boot --description "clean install, no onboarding yet"
```

Dieser Snapshot ist der Rücksetzpunkt auf eine frisch gebootete HAOS-Installation.

---

## 13. Home Assistant Onboarding starten

Im Browser:

```text
http://<jarvis-ip>:8123
```

Dann:

```text
Create my smart home
```

Empfohlene Grundwerte:

```text
Instanzname: Jarvis
Benutzer:    Marc
Sprache:     Deutsch oder Englisch
Zeitzone:    Europe/Luxembourg
Einheiten:   Metric / Celsius / EUR
```

Falls die Installation noch im Arbeitsnetz läuft:

```text
Keine automatisch gefundenen Geräte aus dem Arbeitsnetz dauerhaft übernehmen.
```

Das finale Geräte-Onboarding erfolgt später im Heimnetz.

---

## 14. Zweiten Snapshot nach Onboarding erstellen

Nach abgeschlossenem Onboarding und erstem Zugriff auf das Dashboard:

```text
VM 100 → Snapshots → Take Snapshot
```

Empfohlener Name:

```text
01-jarvis-onboarded-basic
```

Beschreibung:

```text
Home Assistant onboarding completed, no integrations configured yet
```

Wieder:

```text
Include RAM: deaktivieren
```

Alternativ per Shell:

```bash
qm snapshot 100 01-jarvis-onboarded-basic --description "Home Assistant onboarding completed, no integrations configured yet"
```

---

## 15. Feste IP und DNS für Jarvis im Heimnetz

Die feste IP für Jarvis sollte bevorzugt über den Router bzw. DHCP-Server reserviert werden.

Empfehlung:

```text
Hostname: jarvis
FQDN:     jarvis.multiversum.network
IP:       passend zum Heimnetz
```

Beispiel:

```text
jarvis.multiversum.network → 192.168.1.50
```

Der Zugriff erfolgt später über:

```text
http://jarvis.multiversum.network:8123
```

oder:

```text
http://jarvis:8123
```

Wichtig:

```text
Die finale IP wird erst zu Hause gesetzt.
Die aktuelle Arbeitsnetz-IP ist nur temporär.
```

---

## 16. Proxmox-Netzwerk beim Umzug ins Heimnetz beachten

Wenn `pve1` noch im Arbeitsnetz eingerichtet wurde, muss vor oder nach dem Umzug die Proxmox-Netzwerkkonfiguration angepasst werden.

Relevante Dateien:

```bash
cat /etc/network/interfaces
cat /etc/hosts
```

Beispiel für das spätere Heimnetz:

```text
/etc/network/interfaces:
address 192.168.1.40/24
gateway 192.168.1.1

/etc/hosts:
192.168.1.40 pve1.multiversum.network pve1
```

Nach Änderung:

```bash
shutdown now
```

Dann den NUC ins Zielnetz bringen und starten.

---

## 17. Home-Assistant-Backups einrichten

Proxmox-Snapshots sind praktisch, ersetzen aber kein Backup.

In Home Assistant:

```text
Settings → System → Backups
```

Erstes internes Backup erstellen:

```text
01-jarvis-basic-onboarding
```

Dieses Backup liegt innerhalb der HAOS-VM.

Zusätzlich sollte ein externes Backupziel genutzt werden, z. B.:

```text
Synology NFS Share als Proxmox VZDump Backup-Ziel
```

Empfohlene Proxmox-Backup-Struktur:

```text
Storage:     synology-backup
Mode:        Snapshot
Compression: zstd
Retention:   7 daily, 4 weekly, 3 monthly
```

---

## 18. USB-Passthrough für spätere Hardware

Für Zigbee-, Z-Wave- oder Bluetooth-Hardware wird später USB-Passthrough benötigt.

Am Proxmox-Host prüfen:

```bash
lsusb
```

Beispiel:

```text
Bus 003 Device 003: ID 1a86:55d4 Zigbee USB Dongle
```

In der Proxmox-Weboberfläche:

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

Vendor-/Device-ID ist meistens stabiler, falls ein Gerät umgesteckt wird oder sich Port-Nummern ändern.

Wichtig:

```text
USB-Passthrough erst später einrichten.
Die HAOS-Basis soll zuerst stabil laufen.
```

---

## 19. Was jetzt bewusst noch nicht eingerichtet wird

Direkt nach der HAOS-Installation wird noch nicht alles aktiviert.

Noch nicht:

```text
HACS
Zigbee
Z-Wave
Bluetooth
OpenWB
Grocy
Mealie
Frigate
Voice
LLM/KI
Dashboards
Automationen
```

Erst wird eine stabile Jarvis-Basis geschaffen. Danach folgen Integrationen und Dienste Schritt für Schritt.

---

## 20. Zielzustand dieser Anleitung

Nach Abschluss dieser Anleitung gilt:

```text
Proxmox VE läuft auf pve1
HAOS läuft als VM 100
VM-Name: jarvis-haos
Jarvis ist per Browser erreichbar
Erster Snapshot vor Onboarding ist vorhanden
Zweiter Snapshot nach Onboarding ist vorbereitet
finale IP/DNS-Konfiguration erfolgt später im Heimnetz
```

Jarvis ist damit als Home-Assistant-Kern installiert.

Pepper bleibt weiterhin die Assistant-/KI-Schicht.

Jarvis ist die zentrale Home-Automation-Instanz.

---

## Fertig

Damit läuft Home Assistant OS als Jarvis-VM auf Proxmox VE.

Die empfohlene Grundlogik lautet:

```text
Saubere HAOS-VM
+
lokale ZFS-Snapshots
+
externe Backups auf Synology
+
später gezielte Integrationen
```

Erst die stabile Basis, dann die Intelligenz.

Der Butler steht. Die Manieren kommen danach.
