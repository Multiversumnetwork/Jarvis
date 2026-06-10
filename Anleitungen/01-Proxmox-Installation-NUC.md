# Proxmox VE 9.2 auf Intel NUC – Installationsanleitung

> Voraussetzung: Ein bootfähiger USB-Stick mit dem offiziellen Proxmox-VE-ISO ist bereits erstellt.

---

## 1. BIOS/UEFI vorbereiten

NUC einschalten und sofort **F2** drücken, um ins BIOS zu kommen.

- **Secure Boot deaktivieren** (unter "Boot" oder "Security") – vermeidet Bootprobleme mit dem Proxmox-Installer.
  > Proxmox VE 9 kann prinzipiell mit aktiviertem Secure Boot booten. Deaktivieren ist der konservativere Weg – empfohlen.

- **Intel VT-x aktivieren** (unter "Advanced" → "CPU Configuration" → "Intel Virtualization Technology") – **zwingend erforderlich** für das 3-VM-Setup (HAOS / JarvisBrain / Services). Ohne VT-x lassen sich keine VMs starten.
  > Auf den meisten NUCs ist VT-x werkseitig aktiv – jetzt kurz verifizieren ist billiger als ein zweiter BIOS-Teardown.

- **Intel VT-d aktivieren** (unter "Advanced" → "CPU Configuration" → "Intel VT for Directed I/O") – nicht zwingend für den USB-Passthrough der ZBT-2 / ZWA-2 in die HAOS-VM (läuft über QEMU-USB-Redirect), aber Voraussetzung für späteren PCIe-Passthrough. Kostet nichts, einmal setzen und nie wieder dran denken.

- Optional: **Legacy/UEFI Boot** prüfen. Proxmox bootet sauber im UEFI-Modus, das kann so bleiben.
- Änderungen speichern (meist **F10**) und neu starten.

## 2. Vom USB-Stick booten

USB-Stick einstecken, NUC neu starten und während des Intel-Logos **F10** drücken für das Boot-Menü. Dort den USB-Stick auswählen.

## 3. Installer starten

Im Proxmox-Bootmenü **"Install Proxmox VE (Graphical)"** wählen. (Falls die grafische Variante zickt – etwa bei Display-Problemen – gibt es alternativ den Terminal-/Konsolen-Installer.)

Danach die **EULA** akzeptieren.

## 4. Zielfestplatte & Dateisystem (ZFS)

Bei den meisten NUCs ist das die einzige NVMe/SSD. Über **"Options"** das Dateisystem festlegen:

- **Filesystem: `zfs (RAID0)`** auswählen (bei einer einzelnen Platte ist RAID0 die richtige Wahl).
- Standard-Optionen für `ashift`, `compression` (lz4) und `checksum` können i. d. R. übernommen werden.
- **ZFS ARC:** PVE 9.2 begrenzt den ARC-Cache ab Werk auf ~10 % des RAM – kein manuelles Eingreifen erforderlich. Bei Bedarf später über `zfs_arc_max` in `/etc/modprobe.d/zfs.conf` fixieren.

> Vorteil ZFS: Snapshots auf Dateisystemebene, Kompression und Datenintegrität per Checksums – gut für VM-Snapshots und Rollbacks.

## 5. Region & Tastatur

Land (Luxembourg), Zeitzone (Europe/Luxembourg) und Tastaturlayout (German oder Swiss French – je nach Setup) einstellen.

## 6. Root-Passwort & E-Mail

Sicheres Root-Passwort vergeben und eine gültige E-Mail-Adresse eintragen (dorthin gehen später System-Benachrichtigungen).

## 7. Netzwerk konfigurieren

Für einen Server eine **statische IP** verwenden:

- **Management Interface**: die NIC des NUC auswählen
- **Hostname (FQDN)**: `pve1.multiversum.network`
- **IP-Adresse / Netzmaske / Gateway / DNS** entsprechend dem Infrastruktur-Segment eintragen (Readme 4.2)

## 8. Installation abschließen

Zusammenfassung prüfen → **Install**. Nach dem Durchlauf rebootet der NUC automatisch.
**USB-Stick beim Reboot ziehen**, sonst startet der Installer erneut.

## 9. Erster Zugriff

Vom Rechner im selben Netz im Browser aufrufen:

```
https://pve1.multiversum.network:8006
```

(Alternativ direkt per IP: `https://<deine-IP>:8006`)

Login mit Benutzer `root` und gesetztem Passwort. Die Zertifikatswarnung kann akzeptiert werden.

---

## 10. No-Subscription-Repository einrichten (Proxmox VE 9.2)

Ohne gültige Subscription ist das Enterprise-Repo nicht nutzbar und `apt update` schlägt mit `401 Unauthorized` fehl. Stattdessen das kostenlose No-Subscription-Repo aktivieren.

> **Wichtig für PVE 9.x:** Proxmox 9 verwendet das neue **deb822-`.sources`-Format** unter `/etc/apt/sources.list.d/`. Die alten einzeiligen `.list`-Einträge aus PVE 8 (`deb http://...`) gibt es nicht mehr. Suite ist `trixie` (Debian 13).

### Variante A – über die Web-GUI (empfohlen)

1. `Datacenter → <node> → Updates → Repositories`
2. Das **Enterprise-Repo** (`pve-enterprise`) auswählen und auf **Disable** setzen; ebenso `ceph ... enterprise`, falls vorhanden.
3. **Add** klicken → **No-Subscription** auswählen → hinzufügen.

### Variante B – per SSH / Konsole

**Enterprise-Repos deaktivieren** – im deb822-Format geschieht das durch eine `Enabled: false`-Zeile:

```bash
# PVE Enterprise
echo 'Enabled: false' >> /etc/apt/sources.list.d/pve-enterprise.sources

# Ceph Enterprise (falls vorhanden)
echo 'Enabled: false' >> /etc/apt/sources.list.d/ceph.sources
```

**No-Subscription-Repo hinzufügen:**

```bash
cat > /etc/apt/sources.list.d/pve-no-subscription.sources <<'EOF'
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
EOF
```

> Hinweis: `Suites: trixie` entspricht Proxmox VE 9.x (Debian 13). Bei PVE 8.x würde dort `bookworm` stehen. Bei der ISO-Installation ist der Signaturschlüssel (`proxmox-archive-keyring.gpg`) bereits vorhanden.

Anschließend aktualisieren:

```bash
apt update && apt full-upgrade -y
```

### Optional – Subscription-Popup beim Login abschalten

```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" \
  /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js \
  && systemctl restart pveproxy.service
```

> Achtung: Dieser Patch wird bei Updates des `proxmox-widget-toolkit` überschrieben. Empfehlung: stattdessen das Community-Skript aus Abschnitt 11 verwenden – dieses setzt einen persistenten DPkg-Hook.

---

## 11. Komfort-Alternative: Community Post-Install-Skript

Statt die Repos und das Nag-Popup manuell zu konfigurieren (Abschnitt 10), erledigt das **"PVE Post Install"-Skript** der *Proxmox VE Helper-Scripts* alles interaktiv in einem Durchlauf. Es ist der Nachfolger der bekannten tteck-Skripte und wird community-gepflegt.

In der **Proxmox-VE-Shell** (Node → Shell oder per SSH als `root`) ausführen:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```

Das Skript bietet per Dialog u. a. an:

- Enterprise-Repo deaktivieren
- PVE-Sources hinzufügen/korrigieren
- No-Subscription-Repo aktivieren
- (optionales) Test-Repo hinzufügen
- Subscription-Nag-Popup abschalten (persistenter DPkg-Hook – Update-sicher)
- Proxmox VE aktualisieren und neu starten

Empfehlung der Maintainer: die abgefragten Optionen i. d. R. mit **"yes" (y)** bestätigen. Unterstützt werden u. a. PVE 9.2 / Debian 13 Trixie (zuletzt aktualisiert: 8. Juni 2026).

> **Sicherheitshinweis:** Das Skript läuft als `root` auf dem Hypervisor. Vor dem Ausführen kurz die Quelle prüfen (GitHub: `community-scripts/ProxmoxVE`) – das ist bei jedem Curl-to-Bash-Befehl gute Praxis.

---

## Fertig

Damit läuft Proxmox VE 9.2 mit ZFS-Backend und funktionierenden Updates über das No-Subscription-Repo.

**Nächste Schritte:**
- 3-VM-Setup anlegen: HAOS / JarvisBrain / Jarvis Services (→ `02-VM-Setup.md`)
- HAOS installieren + USB-Passthrough ZBT-2 / ZWA-2 (→ `03-HAOS-Installation.md`)
- HA Green als Cold/Warm-Standby konfigurieren (→ Readme Kap. 8.3)