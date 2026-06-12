# 04 – JarvisCore-VM aus Template erstellen

> Ziel: Eine produktive **JarvisCore-VM** aus dem bestehenden Debian-13-Docker-Template erstellen und sauber bis zum Snapshot **before-hermes-install** vorbereiten.
>
> Stand: 2026-06-12

---

## 0. Namensklärung

Die Projekt-README definiert die VM-Struktur wie folgt:

```text
JarvisMansion   = Home Assistant OS / Haussteuerung
JarvisCore      = Agent-/KI-Layer / Hermes Agent
JarvisServices  = Haushalts- & Hilfsdienste
```

Für diese Anleitung gilt daher:

```text
VM-Name:  JarvisCore
Hostname: jarvis-core
FQDN:     jarvis-core.multiversum.network
Rolle:    Agent-/KI-Layer des JARVIS-Systems
```

Der Begriff **JarvisBrain** kann weiterhin als sprechender Rollenname verwendet werden. Der technische VM-Name bleibt aber **JarvisCore**.

---

## 1. Architekturziel

```text
Proxmox VE auf pve1.multiversum.network
├── VM: JarvisMansion
│   └── Home Assistant OS
│
├── VM: JarvisCore
│   └── Hermes Agent
│       ├── JARVIS Persona
│       ├── Memory
│       ├── Skills
│       ├── Home-Assistant-Anbindung
│       └── LLM-Backend remote oder später lokal
│
└── VM: JarvisServices
    └── Mealie, Grocy, weitere Haushaltsdienste
```

Wichtiges Prinzip:

```text
Home Assistant steuert das Haus.
Hermes / JarvisCore analysiert, assistiert und orchestriert.
Die harte Sicherheitsgrenze ist die eigene VM.
```

Home Assistant wird in dieser Anleitung **nicht** verändert.
Hermes wird in dieser Anleitung **noch nicht** installiert.

---

## 2. Voraussetzungen aus dem Template

Das Debian-13-Docker-Template ist bereits vorbereitet.

Vorhanden im Template:

```text
root-User vorhanden
User marc vorhanden
marc besitzt sudo-/root-Rechte
SSH ist eingerichtet
QEMU Guest Agent ist installiert
Docker ist installiert
Docker Compose Plugin ist installiert
Grundlegende Admin-Werkzeuge sind installiert
```

Nicht erneut ausführen:

```text
root-User nicht neu anlegen
User marc nicht neu anlegen
Docker nicht erneut installieren
Docker Compose nicht erneut installieren
keine neue JARVIS Persona erstellen
```

Die JARVIS Persona existiert bereits und wird später in die Hermes-Konfiguration übernommen.

---

## 3. Benutzerkonzept

Für JarvisCore werden zwei Benutzerrollen unterschieden:

```text
marc    = administrativer Benutzer mit sudo-/root-Rechten
jarvis  = dedizierter Runtime-/Service-User für Hermes
```

Begründung:

- `marc` bleibt der Admin- und Wartungsuser.
- Hermes soll später nicht dauerhaft unter dem persönlichen Admin-User laufen.
- `jarvis` ist der saubere, sprechende Dienstbenutzer für den JARVIS-Agenten.
- Die spätere Hermes-Konfiguration liegt dadurch unter `/home/jarvis/.hermes/`.

Wichtig:

```text
Der User jarvis bekommt zunächst keine sudo-Rechte.
Docker-Rechte für jarvis werden erst vergeben, wenn Hermes sie wirklich benötigt.
```

Damit bleibt die VM die harte Sicherheitsgrenze, aber innerhalb der VM läuft Hermes nicht unnötig als Admin-User.

---

## 4. Ziel-VM

Template:

```text
VM-ID: 9000
Name:  debian13-docker-template
```

Neue VM:

```text
VM-ID:    101
Name:     JarvisCore
Hostname: jarvis-core
FQDN:     jarvis-core.multiversum.network
```

Falls VM-ID `101` bereits belegt ist, die nächste freie ID verwenden und die Befehle entsprechend anpassen.

---

## 5. Empfohlene Ressourcen

Startkonfiguration:

```text
CPU:        2 vCPU
RAM:        4 GB
Disk:       32 GB
Netzwerk:   vmbr0
Autostart:  ja
```

Begründung:

- Hermes selbst benötigt keine GPU.
- Das LLM-Backend liegt in Phase 1 remote oder später separat lokal.
- 4 GB RAM reichen für Hermes, Gateway, Skills und Home-Assistant-Anbindung zunächst aus.
- Die VM kann später auf 8 GB RAM erweitert werden.

---

## 6. VM aus Template klonen

Auf dem Proxmox-Host einloggen:

```bash
ssh root@pve1.multiversum.network
```

Full Clone aus dem Debian-Docker-Template erstellen:

```bash
qm clone 9000 101 --name JarvisCore --full true --storage local-zfs
```

Ressourcen setzen:

```bash
qm set 101 --memory 4096 --cores 2
```

Autostart aktivieren:

```bash
qm set 101 --onboot 1
```

VM starten:

```bash
qm start 101
```

---

## 7. Erste Anmeldung

Nach dem Start die IP-Adresse in Proxmox oder im Router ermitteln.

Dann per SSH verbinden:

```bash
ssh marc@<IP-DER-VM>
```

Beispiel:

```bash
ssh marc@192.168.1.101
```

Der Benutzer `marc` stammt bereits aus dem Template und besitzt sudo-/root-Rechte.

---

## 8. Hostname setzen

In der neuen VM:

```bash
sudo hostnamectl set-hostname jarvis-core
```

`/etc/hosts` prüfen:

```bash
sudo nano /etc/hosts
```

Empfohlener Inhalt:

```text
127.0.0.1       localhost
127.0.1.1       jarvis-core.multiversum.network jarvis-core
```

Danach rebooten:

```bash
sudo reboot
```

Nach dem Neustart erneut verbinden:

```bash
ssh marc@jarvis-core
```

Falls DNS noch nicht gesetzt ist, weiterhin per IP verbinden.

---

## 9. Netzwerk / DHCP-Reservierung

Im Router bzw. DHCP-Server eine feste Reservierung setzen:

```text
Hostname: jarvis-core
FQDN:     jarvis-core.multiversum.network
IP:       192.168.1.xxx
```

Die konkrete IP richtet sich nach dem Heimnetz.

Wichtig:

- Keine wechselnde DHCP-Adresse für JarvisCore.
- JarvisCore muss Home Assistant später auf Port `8123` erreichen können.
- Home Assistant bleibt separat als JarvisMansion bestehen.

Test:

```bash
ping -c 4 jarvis-core.multiversum.network
ping -c 4 <HOME-ASSISTANT-IP>
```

Falls DNS noch nicht aktiv ist, testweise per IP arbeiten.

---

## 10. Grundsystem prüfen und aktualisieren

Nach dem Reboot:

```bash
hostnamectl
ip a
df -h
free -h
```

Erwartung:

```text
Static hostname: jarvis-core
```

System aktualisieren:

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

Nach erneutem Login:

```bash
hostnamectl
```

---

## 11. Voraussetzungen für spätere Hermes-Installation prüfen

Hermes wird später nach offizieller Installationsweise installiert.

Für Linux/macOS/WSL2/Termux ist der vorgesehene CLI-Installer:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

Diese Anleitung führt den Installer **noch nicht** aus.

Für die spätere Installation ist laut Hermes-Dokumentation auf Nicht-Windows-Systemen als zentrale Voraussetzung nur `git` notwendig; weitere Bestandteile wie `uv`, Python 3.11, Node.js v22, ripgrep und ffmpeg werden durch den Installer behandelt.

Daher jetzt nur prüfen:

```bash
git --version
curl --version
```

Falls `git` oder `curl` fehlen:

```bash
sudo apt update
sudo apt install -y git curl
```

---

## 12. Docker prüfen

Docker ist im Template bereits installiert. Es wird hier nur geprüft.

Docker-Version prüfen:

```bash
docker --version
```

Docker Compose Plugin prüfen:

```bash
docker compose version
```

Testcontainer starten:

```bash
docker run --rm hello-world
```

Wenn `hello-world` erfolgreich läuft, ist die Docker-Basis einsatzbereit.

Falls Docker nur mit `sudo` funktioniert, Benutzergruppen prüfen:

```bash
groups
```

Der Benutzer `marc` sollte Mitglied der Gruppe `docker` sein.

Falls nicht, ist das Template entsprechend nachzubessern:

```bash
sudo usermod -aG docker marc
sudo reboot
```

Danach erneut testen:

```bash
docker run --rm hello-world
```

Wichtig:

```text
Der spätere Runtime-User jarvis wird hier noch nicht zur docker-Gruppe hinzugefügt.
Docker-Zugriff für Hermes wird erst bewusst freigegeben, wenn wir ihn wirklich brauchen.
```

---

## 13. Runtime-User jarvis anlegen

Der User `jarvis` dient später als dedizierter Hermes-/JARVIS-Runtime-User.

Prüfen, ob er bereits existiert:

```bash
id jarvis
```

Falls der User noch nicht existiert:

```bash
sudo adduser --disabled-password --gecos "JARVIS Runtime User" jarvis
```

Kein sudo einrichten.
Keine docker-Gruppe hinzufügen.

Prüfen:

```bash
id jarvis
sudo -l -U jarvis
```

Erwartung:

```text
jarvis existiert
jarvis hat keine sudo-Rechte
jarvis ist noch nicht Mitglied der Gruppe docker
```

SSH-Zugang für `jarvis` ist für Phase 1 nicht zwingend notwendig. Administration erfolgt weiter über `marc`.

---

## 14. Projektverzeichnis vorbereiten

Das Projektverzeichnis dient nur als sauberer Arbeits- und Dokumentationsort für JarvisCore.

Wichtig:

```text
Hier wird nicht HERMES_HOME vorbereitet.
Die offizielle Hermes-Installation nutzt standardmäßig das Home-Verzeichnis des ausführenden Users.
Für den späteren Runtime-User jarvis wäre das /home/jarvis/.hermes/.
```

Verzeichnis anlegen:

```bash
sudo mkdir -p /opt/jarvis-core
sudo chown -R marc:marc /opt/jarvis-core
cd /opt/jarvis-core
```

Minimale Struktur:

```bash
mkdir -p docs backups
```

Optional eine kurze README anlegen:

```bash
nano /opt/jarvis-core/README.md
```

Inhalt:

```markdown
# JarvisCore

JarvisCore ist die eigene VM für den Agent-/KI-Layer des JARVIS-Systems.

- VM-Name: JarvisCore
- Hostname: jarvis-core
- Rolle: Hermes Agent / JARVIS Core
- Admin-User: marc
- Runtime-User: jarvis
- Home Assistant bleibt separat in JarvisMansion.
- Die harte Sicherheitsgrenze ist die VM.

Hermes wird später nach offizieller Installationsweise installiert.
Die bestehende JARVIS Persona wird später in die Hermes-Konfiguration übernommen.
```

Rechte setzen:

```bash
chmod 755 /opt/jarvis-core
```

---

## 15. Keine Hermes-Struktur manuell anlegen

In dieser Anleitung werden bewusst **keine** Hermes-Konfigurationsdateien angelegt.

Diese Dateien und Verzeichnisse werden jetzt **nicht** erstellt:

```text
/opt/jarvis-core/data/config.yaml
/opt/jarvis-core/data/SOUL.md
/opt/jarvis-core/data/skills/
/opt/data
/home/marc/.hermes/.env
/home/marc/.hermes/config.yaml
/home/marc/.hermes/SOUL.md
/home/jarvis/.hermes/.env
/home/jarvis/.hermes/config.yaml
/home/jarvis/.hermes/SOUL.md
```

Grund:

```text
Hermes wird später nach offizieller Installationsweise eingerichtet.
Die Hermes-Konfiguration entsteht erst bei der Hermes-Installation bzw. Initialisierung.
Vorher legen wir keine .env und keine SOUL.md manuell an.
```

Die spätere Hermes-Struktur liegt bei Installation als User `jarvis` voraussichtlich unter:

```text
/home/jarvis/.hermes/
├── .env
├── config.yaml
├── SOUL.md
├── skills/
├── memories/
├── sessions/
└── logs/
```

Wichtig für die nächste Anleitung:

```text
Falls HERMES_HOME gesetzt wird, muss dieser Pfad exakt zur tatsächlichen Hermes-Konfiguration passen.
Wenn Hermes standardmäßig unter /home/jarvis/.hermes arbeitet, wird kein abweichendes HERMES_HOME gesetzt.
```

Die bestehende JARVIS Persona wird später nach:

```text
/home/jarvis/.hermes/SOUL.md
```

übernommen oder verlinkt.

---

## 16. Zwischenstand prüfen

```bash
hostnamectl
whoami
groups
id jarvis
docker --version
docker compose version
git --version
curl --version
ls -la /opt/jarvis-core
```

Erwartung:

```text
Hostname ist jarvis-core
angemeldeter Admin-User ist marc
marc besitzt sudo-/root-Rechte
jarvis existiert als Runtime-User
jarvis besitzt keine sudo-Rechte
Docker läuft
Docker Compose Plugin läuft
git und curl sind vorhanden
/opt/jarvis-core existiert
Hermes ist noch nicht installiert
Home Assistant ist noch nicht angebunden
```

---

## 17. Proxmox-Snapshot erstellen

An diesem Punkt ist die VM sauber vorbereitet, aber Hermes ist noch nicht installiert.

Auf dem Proxmox-Host:

```bash
qm snapshot 101 before-hermes-install --description "JarvisCore cloned from Debian Docker template, updated, Docker verified, runtime user jarvis created, hostname and project directory prepared. Hermes not installed yet."
```

Alternativ über Proxmox UI:

```text
VM 101 JarvisCore
→ Snapshots
→ Take Snapshot
```

Snapshot-Name:

```text
before-hermes-install
```

Beschreibung:

```text
JarvisCore aus Debian-Docker-Template geklont, aktualisiert, Hostname gesetzt, Docker geprüft, Runtime-User jarvis erstellt, Projektverzeichnis vorbereitet. Hermes noch nicht installiert.
```

---

## 18. Stopppunkt

Bis hier ist nur die VM vorbereitet.

Erledigt:

```text
JarvisCore-VM aus Template geklont
Hostname gesetzt
DHCP-/DNS-Vorbereitung dokumentiert
System aktualisiert
Docker geprüft
git/curl geprüft
Runtime-User jarvis angelegt
/opt/jarvis-core als Projektverzeichnis vorbereitet
keine Hermes-Konfiguration manuell angelegt
Snapshot before-hermes-install erstellt
```

Nicht erledigt:

```text
Hermes noch nicht installiert
HERMES_HOME nicht gesetzt
.env nicht angelegt
SOUL.md nicht angelegt
Home Assistant noch nicht angebunden
JARVIS Persona noch nicht übernommen
Skills noch nicht eingerichtet
```

Nächster Schritt:

```text
05 – Hermes offiziell als User jarvis installieren und initial einrichten
```
