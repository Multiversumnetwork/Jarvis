# Proxmox VE 9.2 auf Intel NUC – Installationsanleitung

> Ziel: Proxmox VE 9.2.x sauber auf einem Intel NUC installieren, mit ZFS als lokalem Storage für Snapshots und vorbereitetem Betrieb als Jarvis-/Home-Assistant-Host.

> Voraussetzung: Ein bootfähiger USB-Stick mit dem offiziellen Proxmox-VE-ISO ist bereits erstellt.

---

## 1. BIOS/UEFI vorbereiten

NUC einschalten und sofort **F2** drücken, um ins BIOS/UEFI zu kommen.

Empfohlene Einstellungen:

- **Secure Boot deaktivieren**  
  Vermeidet Bootprobleme mit dem Proxmox-Installer.

- **UEFI Boot aktiviert lassen**  
  Proxmox bootet sauber im UEFI-Modus. Legacy Boot ist normalerweise nicht nötig.

- **Intel Virtualization Technology / VT-x aktivieren**  
  Wichtig für VMs.

- **Intel VT-d aktivieren**  
  Empfohlen, falls später USB-, PCIe- oder Geräte-Passthrough genutzt werden soll.

- Änderungen speichern, meist mit **F10**, und neu starten.

---

## 2. Vom USB-Stick booten

USB-Stick einstecken, den NUC neu starten und während des Intel-Logos **F10** drücken, um das Boot-Menü zu öffnen.

Dort den USB-Stick auswählen.

---

## 3. Installer starten

Im Proxmox-Bootmenü auswählen:

```text
Install Proxmox VE (Graphical)
```

Falls die grafische Variante Probleme macht, z. B. bei Display- oder Grafikproblemen, alternativ den Terminal-/Konsolen-Installer verwenden.

Danach die **EULA** akzeptieren.

---

## 4. Zielfestplatte & Dateisystem: ZFS

Bei den meisten NUCs ist die interne NVMe/SSD die einzige Zielfestplatte.

Über **Options** das Dateisystem festlegen:

```text
Filesystem: zfs (RAID0)
```

Bei einer einzelnen SSD ist `zfs (RAID0)` die passende Auswahl im Installer.

Standard-Optionen können in der Regel übernommen werden:

- `ashift`: Standard übernehmen
- `compression`: `lz4`
- `checksum`: aktiviert lassen

### Warum ZFS?

ZFS wird hier primär wegen folgender Vorteile genutzt:

- Snapshots auf Dateisystemebene
- Kompression
- Datenintegrität durch Checksums
- saubere Basis für VM-/Container-Disks

### Wichtiger Hinweis

Bei nur einer SSD bietet ZFS RAID0 **keine Redundanz**.

Snapshots sind sehr nützlich gegen fehlerhafte Updates, versehentliche Änderungen oder kaputte Konfigurationen. Sie ersetzen aber **kein Backup**.

Ein Snapshot schützt nicht vor:

- SSD-Ausfall
- komplettem Pool-Defekt
- Hardware-Schaden
- Diebstahl
- Überspannung
- versehentlich gelöschtem oder zerstörtem Host

Für produktive VMs sollte zusätzlich ein externes Backup-Ziel eingerichtet werden, z. B. ein NFS-Share auf einer Synology oder später ein Proxmox Backup Server.

---

## 5. Region & Tastatur

Einstellungen passend zum Standort setzen:

```text
Country: Luxembourg
Time zone: Europe/Luxembourg
Keyboard layout: German oder Swiss French, je nach Tastatur
```

---

## 6. Root-Passwort & E-Mail

Ein sicheres Root-Passwort vergeben und eine gültige E-Mail-Adresse eintragen.

An diese Adresse gehen später System-Benachrichtigungen.

---

## 7. Netzwerk konfigurieren

Für einen Proxmox-Host sollte eine **statische IP-Adresse** verwendet werden.

Im Installer festlegen:

- **Management Interface**: Netzwerkkarte des NUC auswählen
- **Hostname (FQDN)**: z. B. `pve1.multiversum.network`
- **IP-Adresse / Netzmaske**: passend zum Zielnetz
- **Gateway**: Router-IP
- **DNS Server**: Router-IP oder ein externer DNS-Server

Beispiel:

```text
Hostname: pve1.multiversum.network
IP:       192.168.1.40/24
Gateway:  192.168.1.1
DNS:      192.168.1.1
```

Empfehlung:

- IP-Adresse entweder außerhalb des DHCP-Pools vergeben
- oder im Router per DHCP-Reservierung fest zuweisen
- Hostname vor der produktiven Nutzung final festlegen

---

## 8. Installation abschließen

Zusammenfassung prüfen und die Installation starten.

Nach dem Installationsdurchlauf rebootet der NUC automatisch.

Wichtig:

```text
USB-Stick beim Reboot entfernen.
```

Sonst startet der Installer erneut.

---

## 9. Erster Zugriff auf die Weboberfläche

Von einem Rechner im selben Netzwerk im Browser aufrufen:

```text
https://<deine-IP>:8006
```

Beispiel:

```text
https://192.168.1.40:8006
```

Login:

```text
User: root
Realm: Linux PAM standard authentication
Password: gesetztes Root-Passwort
```

Die Zertifikatswarnung im Browser kann akzeptiert werden.

---

## 10. Community Post-Install-Skript ausführen

Nach der Grundinstallation wird das **PVE Post Install**-Skript der Proxmox VE Helper-Scripts verwendet.

Es erledigt interaktiv u. a.:

- Enterprise-Repository deaktivieren
- No-Subscription-Repository aktivieren
- PVE-Sources korrigieren
- optionales Test-Repository abfragen
- Subscription-Hinweisfenster abschalten
- Proxmox VE aktualisieren
- Neustart anbieten

In der **Proxmox-VE-Shell** ausführen:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```

Empfehlung:

Die abgefragten Standardoptionen können für ein privates Homelab in der Regel mit **yes (y)** bestätigt werden.

Wichtig:

Das Skript läuft als `root` auf dem Hypervisor. Vor dem Ausführen sollte die Quelle bewusst geprüft werden. Curl-to-Bash ist bequem, aber auf einem Hypervisor immer Vertrauenssache.

Nach Updates und eventuellem Neustart prüfen:

```bash
pveversion -v
zpool status
zfs list
```

Der ZFS-Pool sollte den Zustand melden:

```text
state: ONLINE
```

---

## 11. Netzwerk nachträglich ändern / Umzug in ein anderes Netzwerk

Falls Proxmox zunächst in einem anderen Netzwerk installiert wurde, z. B. auf der Arbeit, kann die spätere Heimnetz-IP vor dem Umzug eingetragen werden.

Wichtig: Sobald die neue IP-Konfiguration aktiviert wird, ist die Weboberfläche im aktuellen Netzwerk eventuell nicht mehr erreichbar. Das ist normal.

### Variante A – über die Weboberfläche

Pfad:

```text
Node pve1 → System → Network
```

Dort die Bridge `vmbr0` bearbeiten.

Beispiel für das Heimnetz:

```text
Address: 192.168.1.40/24
Gateway: 192.168.1.1
```

Danach:

```text
Apply Configuration
```

### DNS prüfen

Pfad:

```text
Node pve1 → System → DNS
```

Beispiel:

```text
Search domain: multiversum.network
DNS server 1: 192.168.1.1
DNS server 2: 1.1.1.1
```

### /etc/hosts prüfen

In der Shell:

```bash
cat /etc/hosts
```

Die Heimnetz-IP sollte zum Hostnamen passen:

```text
127.0.0.1       localhost
192.168.1.40    pve1.multiversum.network pve1
```

Falls dort noch die alte IP steht:

```bash
nano /etc/hosts
```

Nach der Änderung sauber herunterfahren:

```bash
shutdown now
```

Dann den NUC ins Zielnetz bringen, per LAN anschließen und starten.

Der Zugriff erfolgt anschließend über die neue IP:

```text
https://192.168.1.40:8006
```

Später kann im lokalen DNS oder Router zusätzlich gesetzt werden:

```text
pve1.multiversum.network → 192.168.1.40
```

---

## 12. Storage prüfen

Nach der Installation in der Proxmox-Weboberfläche prüfen:

```text
Datacenter → Storage
```

Typischerweise vorhanden:

```text
local       → ISO-Images, Container-Templates, Backups
local-zfs   → VM-Disks und Container-Disks
```

Zusätzlich prüfen:

```bash
zpool status
zfs list
```

---

## 13. Backup-Ziel vorbereiten

Vor produktiver Nutzung sollte ein externes Backup-Ziel eingerichtet werden.

Für den Start bietet sich eine Synology per NFS an:

```text
Proxmox VE → VZDump Backup → Synology NFS Share
```

Empfohlene erste Struktur:

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

Danach Backup-Job anlegen:

```text
Datacenter → Backup → Add
```

Empfohlener Startwert:

```text
Schedule: täglich nachts
Mode:     Snapshot
Compression: zstd
Retention: 7 daily, 4 weekly, 3 monthly
```

Langfristig kann ein Proxmox Backup Server ergänzt werden. Für den Anfang ist ein Synology-NFS-Ziel deutlich besser als nur lokale Snapshots.

---

## 14. Nächster Schritt: Home Assistant OS / Jarvis-VM

Nach abgeschlossener Proxmox-Basisinstallation, Updates und Backup-Vorbereitung kann die erste produktive VM erstellt werden.

Für Jarvis:

```text
VM 100: Home Assistant OS
Name:   jarvis-haos
```

Empfohlene Startwerte:

```text
CPU:        2–4 vCPU
RAM:        4 GB
Disk:       32–64 GB
BIOS:       OVMF / UEFI
Machine:    q35
SCSI:       VirtIO SCSI single
Network:    VirtIO, Bridge vmbr0
```

Später kann erweitert werden, z. B. durch:

```text
VM 101: Debian Service Stack
```

für Dienste, die bewusst außerhalb von Home Assistant OS laufen sollen.

---

## Fertig

Damit läuft Proxmox VE 9.2.x auf dem Intel NUC mit ZFS-Backend, funktionierenden Updates über das Community-Post-Install-Skript und vorbereiteter Struktur für Jarvis/Home Assistant.

Die empfohlene Grundlogik lautet:

```text
Lokale ZFS-Snapshots
+
externe Backups auf Synology
+
später optional Proxmox Backup Server
```
