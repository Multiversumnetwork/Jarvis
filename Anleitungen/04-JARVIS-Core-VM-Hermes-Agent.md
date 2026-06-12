# 04 – JarvisCore-VM mit Hermes Agent

> Ziel: Eine produktive **JarvisCore-VM** aus dem bestehenden Debian-13-Docker-Template erstellen und als Basis für den späteren Hermes-Agent-Betrieb vorbereiten.
>
> Stand: 2026-06-12

---

## 0. Namensklärung

Die README verwendet an mehreren Stellen beide Begriffe:

- **JarvisCore** – in der VM-Struktur als Agent-/KI-Layer
- **JarvisBrain** – in der Netzwerk-/Backup-Beschreibung als sprechender Rollenname

Für die konkrete Proxmox-VM wird in dieser Anleitung der technische VM-Name verwendet:

```text
JarvisCore
```

Empfohlener Hostname:

```text
jarvis-core
```

Empfohlener FQDN:

```text
jarvis-core.multiversum.network
```

Rollenname im Systemkonzept:

```text
JarvisBrain = Funktion / Rolle
JarvisCore  = VM / technischer Name
```

Damit bleibt die README-Logik erhalten, aber die VM bekommt einen eindeutigen Namen.

---

## 1. Zielbild

Die JarvisCore-VM ist der getrennte Agent-/KI-Layer des JARVIS-Systems.

```text
Proxmox VE auf Intel NUC
├── VM: JarvisMansion / HAOS
│   └── Home Assistant OS
│
├── VM: JarvisCore
│   └── Hermes Agent
│       ├── Memory
│       ├── Skills
│       ├── Home-Assistant-Anbindung
│       └── LLM-Backend remote oder später lokal
│
└── VM: JarvisServices
    └── Mealie, Grocy, weitere Haushaltsdienste
```

Wichtiges Architekturprinzip:

```text
Home Assistant steuert das Haus.
Hermes analysiert, assistiert und orchestriert.
Die harte Sicherheitsgrenze ist die VM.
```

Hermes läuft in dieser VM per Docker/Compose. Die Kommunikation mit Home Assistant erfolgt später ausschließlich über die Hermes-Home-Assistant-Integration bzw. Home-Assistant-API.

---

## 2. Voraussetzungen

Vorhanden:

```text
Proxmox VE läuft auf pve1.multiversum.network
Debian-13-Docker-Template ist erstellt
Docker und Docker Compose sind im Template installiert
Home Assistant OS läuft separat als JARVIS / JarvisMansion
```

Template laut bisheriger Struktur:

```text
VM 9000: debian13-docker-template
```

Bereits vorhandene HAOS-VM:

```text
VM 100: jarvis-haos
```

Neue VM:

```text
VM 101: JarvisCore
Hostname: jarvis-core
FQDN: jarvis-core.multiversum.network
```

Falls VM-ID 101 bereits belegt ist, die nächste freie ID verwenden.

---

## 3. Empfohlene Ressourcen

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
- 4 GB RAM reichen für Hermes, Gateway, Skills und HA-Anbindung zunächst aus.
- Die VM kann später auf 8 GB RAM erweitert werden.

---

## 4. VM aus Template klonen

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

## 5. Erste Anmeldung

Nach dem Start die IP-Adresse in Proxmox oder im Router ermitteln.

Dann per SSH verbinden:

```bash
ssh marc@<IP-DER-VM>
```

Beispiel:

```bash
ssh marc@192.168.1.101
```

Falls der Benutzer im Template anders heißt, entsprechend anpassen.

---

## 6. Hostname setzen

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

## 7. Netzwerk / DHCP-Reservierung

Im Router bzw. DHCP-Server eine feste Reservierung setzen:

```text
Hostname: jarvis-core
FQDN:     jarvis-core.multiversum.network
IP:       192.168.1.xxx
```

Die konkrete IP richtet sich nach dem Heimnetz.

Wichtig:

- Keine wechselnde DHCP-Adresse für JarvisCore.
- Home Assistant muss JarvisCore später zuverlässig erreichen können.
- JarvisCore muss Home Assistant auf Port `8123` erreichen können.

Test:

```bash
ping -c 4 jarvis-core.multiversum.network
ping -c 4 <HOME-ASSISTANT-IP>
```

---

## 8. Grundsystem prüfen

Nach dem Reboot:

```bash
hostnamectl
ip a
df -h
free -h
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

Erwartung:

```text
Static hostname: jarvis-core
```

---

## 9. Docker prüfen

Docker-Version prüfen:

```bash
docker --version
```

Docker Compose prüfen:

```bash
docker compose version
```

Testcontainer starten:

```bash
docker run --rm hello-world
```

Wenn `hello-world` erfolgreich läuft, ist die Docker-Basis einsatzbereit.

Falls Docker nur mit `sudo` funktioniert, Benutzerrechte prüfen:

```bash
groups
```

Der Benutzer sollte Mitglied der Gruppe `docker` sein.

Falls nicht:

```bash
sudo usermod -aG docker $USER
sudo reboot
```

---

## 10. Verzeichnisstruktur vorbereiten

Arbeitsverzeichnis anlegen:

```bash
sudo mkdir -p /opt/jarvis-core
sudo chown -R $USER:$USER /opt/jarvis-core
cd /opt/jarvis-core
```

Zielstruktur:

```text
/opt/jarvis-core/
├── docker-compose.yml
├── .env
└── data/
    ├── config.yaml
    ├── SOUL.md
    ├── memories/
    ├── skills/
    ├── cron/
    ├── sessions/
    └── logs/
```

Verzeichnisse vorbereiten:

```bash
mkdir -p data/{memories,skills,cron,sessions,logs}
```

Rechte setzen:

```bash
chmod 700 /opt/jarvis-core
chmod 700 /opt/jarvis-core/data
```

---

## 11. Platzhalter für Hermes-Umgebung anlegen

Noch keine produktiven Tokens eintragen.

`.env` anlegen:

```bash
nano /opt/jarvis-core/.env
```

Inhalt:

```env
# JarvisCore / Hermes Agent
TZ=Europe/Luxembourg

# Hermes Home inside the container / runtime
HERMES_HOME=/opt/data

# LLM provider - später setzen
# OPENROUTER_API_KEY=
# ANTHROPIC_API_KEY=
# OPENAI_API_KEY=

# Home Assistant - später setzen
# HASS_URL=http://<HOME-ASSISTANT-IP>:8123
# HASS_TOKEN=
```

Dateirechte setzen:

```bash
chmod 600 /opt/jarvis-core/.env
```

Hinweis:

- Geheimnisse gehören in `.env`.
- Nicht in Git committen.
- Nicht in Screenshots zeigen.

---

## 12. Minimale JARVIS-Konfig vorbereiten

`config.yaml` anlegen:

```bash
nano /opt/jarvis-core/data/config.yaml
```

Startinhalt:

```yaml
# JarvisCore / Hermes Agent
# Phase 1: VM und Hermes-Basis vorbereiten

terminal:
  backend: docker
  timeout: 180
  docker_image: "nikolaik/python-nodejs:python3.11-nodejs20"
  docker_mount_cwd_to_workspace: false
  docker_run_as_host_user: false
  container_cpu: 1
  container_memory: 2048
  container_persistent: true

# Home Assistant Gateway wird später gezielt aktiviert.
# Ohne Filter werden laut Hermes-Doku keine Events weitergereicht.
platforms:
  homeassistant:
    enabled: false
    extra:
      watch_domains:
        - climate
        - binary_sensor
        - alarm_control_panel
      watch_entities: []
      ignore_entities:
        - sensor.uptime
        - sensor.cpu_usage
        - sensor.memory_usage
      cooldown_seconds: 30

updates:
  pre_update_backup: true
  backup_keep: 5
```

Wichtig:

```text
In Phase 1 bleibt Home Assistant deaktiviert.
Erst Hermes installieren und lokal testen.
Danach HASS_URL und HASS_TOKEN setzen.
Danach HA-Verbindung testen.
```

---

## 13. SOUL.md als Platzhalter anlegen

```bash
nano /opt/jarvis-core/data/SOUL.md
```

Inhalt:

```markdown
# JARVIS Core Identity

Du bist JARVIS, der lokale Agent- und Analyse-Layer des Multiversum-Hauses.

Grundsätze:

- Stabilität vor Spielerei.
- Lokal vor Cloud, soweit praktisch möglich.
- Home Assistant bleibt die ausführende Haussteuerung.
- Du analysierst zuerst, handelst danach.
- Kritische Aktionen benötigen explizite Bestätigung.
- Keine Tür-, Schloss-, Garagen-, Alarm- oder sicherheitsrelevanten Aktionen ohne Freigabe.
- Schreibende Home-Assistant-Aktionen sollen bevorzugt über explizite Skripte `script.jarvis_*` erfolgen.
- Bei Unsicherheit: Zustand melden, Vorschlag machen, nicht handeln.

Du bist Infrastruktur, kein kreativer Chat-Charakter.
```

---

## 14. Zwischenstand prüfen

```bash
cd /opt/jarvis-core
find . -maxdepth 3 -type d -print
ls -la
ls -la data
```

Erwartung:

```text
/opt/jarvis-core existiert
.env existiert und ist chmod 600
data/config.yaml existiert
data/SOUL.md existiert
data/skills existiert
data/memories existiert
data/logs existiert
```

---

## 15. Proxmox-Snapshot erstellen

An diesem Punkt ist die VM sauber vorbereitet, aber Hermes ist noch nicht produktiv verbunden.

Auf dem Proxmox-Host:

```bash
qm snapshot 101 before-hermes-install --description "JarvisCore cloned from Debian Docker template, updated, Docker verified, base directories prepared."
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
JarvisCore aus Debian-Docker-Template geklont, aktualisiert, Docker geprüft, Basisstruktur vorbereitet.
```

---

## 16. Stopppunkt

Bis hier ist nur die VM vorbereitet.

Noch nicht erledigt:

```text
Hermes installieren
Hermes starten
LLM-Backend verbinden
Home Assistant Token setzen
Home Assistant Gateway aktivieren
Skills installieren
JARVIS produktiv testen
```

Bewusster Stopppunkt:

```text
Die VM-Grenze steht.
Docker funktioniert.
JarvisCore ist vorbereitet.
Home Assistant bleibt unangetastet.
```

---

## 17. Nächster Schritt

Nächste Anleitung:

```text
05 – Hermes Agent auf JarvisCore installieren und lokal testen
```

Darin folgen:

- Hermes Docker/Compose-Setup
- erster Start
- Logs prüfen
- Hermes-Konfiguration prüfen
- LLM-Backend setzen
- lokaler Funktionstest ohne Home Assistant
- erst danach Home-Assistant-Anbindung über `HASS_URL` und `HASS_TOKEN`

---

## Quellen / Referenz

- Projekt-README: JARVIS v1.8 Foundation Draft
- Template-Anleitung: `03-VM-Template.md`
- Hermes Configuration: https://hermes-agent.nousresearch.com/docs/user-guide/configuration
- Hermes Home Assistant Integration: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/homeassistant
