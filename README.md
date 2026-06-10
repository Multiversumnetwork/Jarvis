# 🌌 JARVIS
# Haussteuerung & Smart-Home-System des Multiversums
## Version 1.8 – Foundation Draft

---

# 🎯 Ziel dieses Dokuments

Dieses Dokument definiert:
- Aufbau
- Zielsetzung
- Hardware
- Sicherheitsarchitektur
- Netzwerkstruktur
- Sprachsystem
- KI-Integration
- Geräteverwaltung
- Dashboard-Struktur
- Backup & Resilienz
- Monitoring & Benachrichtigungen
- Konventionen
- Zukunftsplanung

des JARVIS-Systems innerhalb des Multiversum.network.

JARVIS dient als zentrales Haussteuerungs- und Automatisierungssystem des Multiversums.

---

# 0. 🧭 Designprinzipien & Nicht-Ziele

---

## 0.1 Designprinzipien

- Lokal vor Cloud
- Stabilität vor Spielerei
- Modularität vor Monolith
- Datenschutz vor Komfort
- Dokumentation vor Komplexität
- Automatisierung mit menschlicher Kontrolle
- **Resilienz vor Abhängigkeit:** Jedes integrierte Subsystem muss ohne JARVIS grundfunktionsfähig bleiben

---

## 0.2 Nicht-Ziele

JARVIS ist bewusst **NICHT**:

- kreativer KI-Assistent (das sind Pepper, Raymond, Karen etc.)
- Persona-System mit Charakterspiel
- Cloud-only-System
- Plattform für sicherheitskritische Aktionen ohne explizite Bestätigung
- Ersatz für funktionierende Standalone-Steuerungen (Hue Bridge, Thermostate, Wallbox-UI etc.)

JARVIS ist Infrastruktur, kein Charakter.

---

## 0.3 Resilienz-Prinzip

Jedes integrierte System muss eine sinnvolle Stand-alone-Funktion behalten, falls JARVIS ausfällt:

- Hue Bridge Pro funktioniert weiterhin per App und Schalter
- Heizung muss manuell oder per Standard-Thermostat steuerbar bleiben
- Wallbox lädt im definierten Default-Modus auch ohne JARVIS
- Bewässerung darf bei Ausfall nicht „dauerhaft an" oder „dauerhaft aus" stehen bleiben
- Kameras nehmen über UniFi Protect lokal weiter auf
- **Bei Ausfall des Proxmox-Hosts (NUC) übernimmt der Home Assistant Green als Cold/Warm-Standby die Kernsteuerung (siehe 3.1, 8.3)**

---

# 1. 🧠 Definition von JARVIS

---

## 1.1 Was ist JARVIS?

JARVIS ist das lokale digitale Hausbetriebssystem des Multiversums.

Basis:
- Home Assistant

JARVIS übernimmt:
- Haussteuerung
- Automatisierung
- Sprachsteuerung
- Sensorik
- Energieverwaltung
- Dashboard-Visualisierung
- Geräteintegration
- Alltagsunterstützung

JARVIS ist kein kreativer KI-Assistent wie Pepper oder Raymond.

Der Fokus liegt auf:
- Stabilität
- Steuerung
- Infrastruktur
- Alltag
- Automatisierung
- lokaler Kontrolle

---

# 2. 🎯 Zielsetzung

---

## 2.1 Kurzfristige Ziele

- Aufbau eines stabilen Home-Assistant-Systems
- Sprachsteuerung über lokale Sprachverarbeitung
- Lichtsteuerung über Philips Hue Bridge Pro
- Küchen-Dashboard
- Einkaufslisten
- Müllkalender
- Lagerverwaltung
- Mobile Zugriffe
- Netzwerksegmentierung vorbereiten
- Integration von Synology / Jellyfin vorbereiten
- Vorbereitung von OpenWB und Gartenintegration
- Backup-Strategie auf Synology
- USV für Kernsysteme

---

## 2.2 Mittelfristige Ziele

- Vollständige Hausautomatisierung
- KI-gestützte Assistenzfunktionen
- Energieoptimierung
- Präsenzautomationen über mmWave-Sensorik
- intelligente Routinen
- Gartenautomatisierung
- Kameraintegration über UniFi Protect (+ optional Frigate)
- Mediensteuerung über Jellyfin
- Smarty / P1-Anbindung für Energiedaten
- Monitoring & Alerting etabliert

---

## 2.3 Langfristige Ziele

- Vollständiges Energie-Management
- KI-gestützte Optimierung
- intelligente Verbrauchssteuerung
- zentrale Steuerung aller Infrastrukturkomponenten
- vollständiges digitales Hausbetriebssystem
- Integration von Heizung, PV und Energiespeicher
- optionales lokales LLM auf Proxmox als Conversation Agent

---

# 3. 🖥️ Hardware

---

## 3.1 Zentrale Steuerung

### Intel NUC – Proxmox VE Host

Verwendungszweck:
- Virtualisierungsbasis des gesamten JARVIS-Kerns
- Host für drei getrennte VMs
- saubere Modul-Trennung gemäß „Modularität vor Monolith" (0.1)
- VM-Snapshots & Rollback für risikoarme Änderungen

VM-Struktur:

| VM | Zweck | Inhalt |
|---|---|---|
| **HAOS** | Haussteuerung | Home Assistant OS (offiziell als VM unterstützt) |
| **JarvisBrain** | Agent-/KI-Layer | Hermes Agent (Nous Research); LLM-Backend remote oder lokal (siehe 5.5) |
| **Jarvis Services** | Haushalts- & Hilfsdienste | Mealie, Grocy, weitere Container |

Hinweise:
- HAOS als VM bleibt vollwertige Steuerungsplattform — keine „Spielerei", sondern offiziell unterstütztes Setup.
- KI-Trennung (6.1) wird durch die eigene JarvisBrain-VM hart durchgesetzt: Ein Absturz des Agents berührt die HAOS-Steuerung nicht.
- Grocy-Betrieb in der Services-VM (SQLite + Docker bleiben unverändert).

### Home Assistant Green – Cold/Warm-Standby

Verwendungszweck:
- Ausfallsicherung für den Single-Hardware-Punkt NUC (Resilienz-Prinzip 0.3)
- Restore-Ziel für das nächtliche HA-Backup (siehe 8.3)

Stand-alone-Verhalten:
- Bei NUC-Totalausfall: Green booten, letztes Synology-Backup einspielen, Kernsteuerung wiederherstellen.
- Green wird nicht produktiv parallel betrieben, sondern als Standby vorgehalten.

---

## 3.2 Sprachsteuerung

### Home Assistant Voice Preview Edition

Verwendungszweck:
- lokale Sprachsteuerung
- Wake Word Detection
- Sprachdialoge
- Steuerung von Automationen

---

## 3.2.1 Sprachsatelliten

Geplant:
- 2–3 zusätzliche Sprachsatelliten
- lokale Sprachsteuerung in mehreren Räumen
- Integration in Home Assistant Assist

Geplante Standorte:
- Schlafzimmer
- Lounge / Wohnzimmer
- Studio / Garage

Funktionen:
- Wake Word Detection
- lokale Sprachsteuerung
- Raumkontext
- Mediensteuerung
- spätere Intercom-Funktionen

---

## 3.3 Netzwerk & Infrastruktur

Vorhandene Systeme:
- FritzBox
- VOO-Modem mit fester IP
- Multiversum CoreStack
- Authentik
- **Intel NUC mit Proxmox VE (JARVIS-Kern)**

Geplante Systeme:
- UniFi Netzwerk-Infrastruktur
- UniFi Protect
- VLAN-Struktur

---

## 3.4 Küchen-Dashboard

Geplant:
- fest installiertes Tablet
- zentrales JARVIS Dashboard
- Hausstatus
- Lichtsteuerung
- Heizungsstatus
- Einkaufslisten
- Müllkalender
- Gartenübersicht
- Kamerazugriff
- Energieübersicht
- Medienübersicht

---

## 3.5 Medien- & Entertainment-Systeme

### Synology DS918+

NAS-System auf Basis von DSM 7 mit Jellyfin als Medienserver.

Verwendungszwecke:
- zentrale Medienablage
- Jellyfin-Medienserver
- Streaming
- Archivierung
- interne Medienbereitstellung
- Ziel für nächtliche JARVIS-Backups
- optional: Hosting von Frigate / Uptime Kuma in Docker

---

### Jellyfin

Domain:
- `cinema.multiversum.network`

Funktionen:
- Filme
- Serien
- Dokumentationen
- Medienarchiv
- Streaming innerhalb des Multiversums

Spätere Integration in JARVIS:
- Dashboard-Anzeige
- Mediensteuerung
- Sprachsteuerung
- Szenen & Automationen

---

## 3.6 Unterbrechungsfreie Stromversorgung (USV)

Geplant:
- USV für Kernsysteme (Modell TBD, z. B. APC Back-UPS Pro 900 oder vergleichbar)

Abgesicherte Komponenten:
- Intel NUC (Proxmox-Host)
- FritzBox
- UniFi-Core-Switch
- ggf. Home Assistant Green (Standby)
- ggf. Synology DS918+

Funktionen:
- Überbrückung kurzer Stromausfälle
- sauberes Herunterfahren bei längerem Ausfall
- NUT-Integration in Home Assistant für Status & Automationen
- Benachrichtigung bei Netzausfall

---

## 3.7 Funkadapter & Koordinatoren

Neben der Philips Hue Bridge Pro (eigener, isolierter Zigbee-Stack für Hue-Licht) kommen die offiziellen Home-Assistant-Funkadapter zum Einsatz:

| Adapter | Protokoll | Aufgabe |
|---|---|---|
| Philips Hue Bridge Pro | Zigbee (Hue) | dediziert für Hue-Licht, läuft stand-alone weiter (12.1) |
| Home Assistant Connect ZBT-2 | Zigbee / Thread / Matter | allgemeiner Zigbee-/Thread-Koordinator für JARVIS |
| Home Assistant Connect ZWA-2 | Z-Wave 800 (EU 868,42 MHz) | Z-Wave-Koordinator inkl. Z-Wave Long Range |

Hinweise:
- ZBT-2 ist der offizielle Nachfolger des eingestellten ZBT-1 (SkyConnect); ein Adapter betreibt entweder Zigbee **oder** Thread.
- ZWA-2 (Z-Wave 800) unterstützt Z-Wave und Z-Wave Long Range gleichzeitig; EU-Variante wegen 868,42-MHz-Frequenz erforderlich.
- Bewusste Trennung: Hue bleibt auf eigener Bridge, damit Lichtsteuerung auch bei JARVIS-Ausfall funktioniert (0.3).
- Adapter werden direkt am Proxmox-Host (USB-Passthrough in die HAOS-VM) betrieben.

---

# 4. 🌐 Netzwerkarchitektur

---

## 4.1 Grundstruktur

```text
Internet
   │
VOO Modem
   │
FritzBox
   │
─────────────────────
Internes Netzwerk
─────────────────────
   │
Intel NUC (Proxmox VE)
   │
├── VM: HAOS (JARVIS / Home Assistant)
├── VM: JarvisBrain (Hermes Agent)
├── VM: Jarvis Services (Mealie, Grocy, ...)
   │
JARVIS (Home Assistant)
   │
├── Zigbee / Thread / Matter / Z-Wave
├── Voice-System
├── Dashboard-Tablet
├── Hue Bridge Pro
├── Kameras / UniFi Protect
├── Garten / Navimow
├── Garage / OpenWB
├── Synology / Jellyfin
├── Wetterstation / Ecowitt
├── Smarty / P1-Reader
└── lokale Dienste
```

---

## 4.2 Netzwerksegmente

| Netzwerk | Aufgabe |
|---|---|
| Infrastruktur | Server & Core-Systeme |
| Smart Home | Home Assistant & zentrale Smart-Home-Komponenten |
| IoT | Sensoren, Aktoren, Haushaltsgeräte, Verbraucher |
| Kameras | UniFi Protect und spätere Kamerasysteme |
| Medien | Synology DS918+ und Jellyfin |
| Gäste | isolierter Gastzugang |
| Produktion | Produktionssysteme der Moschterfabrick |

---

## 4.3 Remote-Zugriff

Zugriff erfolgt nicht direkt per offenem Port auf Home Assistant.

Geplanter Zugriff:
- Reverse Proxy
- Authentik
- Tailscale
- interne Sicherheitsrichtlinien

Domain:
- `jarvis.multiversum.network`

---

# 5. 🎙️ Sprachsystem

---

## 5.1 Zielsetzung

JARVIS soll vollständig lokal sprachgesteuert nutzbar sein.

Fokus:
- schnelle Reaktionszeiten
- Datenschutz
- lokale Verarbeitung
- natürliche Steuerung

---

## 5.2 Spracharchitektur

| System | Aufgabe |
|---|---|
| OpenWakeWord | Wake Word Detection |
| Whisper | Speech-To-Text |
| Piper | Text-To-Speech |
| Home Assistant Assist | Sprachlogik |
| Conversation Agent | natürlichsprachige Steuerung (siehe 5.5) |

---

## 5.3 Wake Word

Geplante Wake Words:
- „Hey Jarvis"
- „Jarvis"

---

## 5.4 Sprachcharakter

Die Stimme von JARVIS soll:
- deutschsprachig
- technisch
- ruhig
- präzise
- sachlich
- souverän

wirken.

Keine übertrieben emotionale oder verspielte Sprachcharakteristik.

---

## 5.5 Conversation Agent

Über reine Intent-Matches hinaus benötigt Assist für natürlichsprachige Steuerung
einen Conversation Agent. Ab v1.6 übernimmt diese Rolle der **Hermes Agent**
(Nous Research) in der **JarvisBrain-VM**.

Hermes ist der Agent-/Orchestrierungslayer mit Memory, Skills, MCP und
nativer Home-Assistant-Anbindung (REST-Tools zum Abfragen und Steuern von
Entitäten). Hermes bringt **selbst kein Modell** mit, sondern zeigt auf ein
austauschbares LLM-Backend — das entkoppelt die Hardwarefrage von der Agent-Wahl.

| Komponente | Rolle |
|---|---|
| HA Assist (Intent-basiert) | Fallback, vollständig lokal, keine Abhängigkeit |
| Hermes Agent (JarvisBrain-VM) | Agent-Layer: Memory, Skills, HA-Tools, MCP |
| LLM-Backend (austauschbar) | DeepSeek via Infomaniak (EU/CH) **oder** lokales Ollama |

Backend-Strategie:
- **Phase 1–2:** HA Assist intent-basiert als Basis.
- **Phase 3:** Hermes Agent mit **remote LLM-Backend** (DeepSeek via Infomaniak). Der NUC muss keine GPU-Inferenz stemmen — passt zur Hardware ohne dedizierte GPU.
- **Phase 4+:** Optional Umschwenk auf **lokales Ollama**, sobald GPU-fähige Hardware verfügbar ist. Konfiguration bleibt gleich, nur der Endpoint wechselt.

Prinzip: „Lokal vor Cloud" (0.1) bleibt das Ziel; das remote Backend ist eine
bewusste Übergangslösung, kein Endzustand.

---

# 6. 🤖 KI-Integration

---

## 6.1 Grundprinzip

Die KI-Funktionen werden bewusst von der eigentlichen Haussteuerung getrennt.

Home Assistant bleibt:
- Steuerungsplattform
- Geräteplattform
- Automatisierungssystem

KI-Systeme werden modular angebunden.

---

## 6.2 KI-Architektur

Geplant:
- **Hermes Agent (Nous Research)** als Agent-Runtime in der JarvisBrain-VM
- LLM-Backend austauschbar: DeepSeek (Infomaniak) → später lokales Ollama
- API-basierte KI-Integration
- lokale Sprachverarbeitung
- KI-gestützte Assistenzfunktionen
- Agent-Layer strikt von der HAOS-Steuerung getrennt (eigene VM, vgl. 6.1)

Nicht geplant:
- lokales LLM auf dem (ehemaligen) Home Assistant Green
- Vermischung von JARVIS mit kreativen KI-Personas

---

## 6.3 Ziele der KI-Integration

KI soll unterstützen bei:
- intelligenten Routinen
- Sprachdialogen
- Analyse
- Empfehlungen
- Energieoptimierung
- Alltagsunterstützung

Die finale Kontrolle bleibt beim Menschen.

---

## 6.4 KI-Grundprinzipien

- lokale Steuerung bleibt priorisiert
- KI dient als Assistenzsystem
- keine sicherheitskritischen Aktionen ohne definierte Regeln
- finale Kontrolle bleibt beim Menschen
- Home Assistant bleibt auch ohne KI funktionsfähig

---

# 7. 🔐 Sicherheitsarchitektur

---

## 7.1 Grundprinzipien

- keine unnötigen offenen Ports
- lokale Kommunikation bevorzugt
- zentrale Authentifizierung
- Netzwerksegmentierung
- minimale externe Angriffsfläche
- Home Assistant bleibt intern primär lokal erreichbar

---

## 7.2 Authentifizierung

Externer Zugriff erfolgt über:
- Reverse Proxy
- Authentik
- Tailscale

Home Assistant bleibt intern lokal erreichbar.

Authentik dient als äußere Zugriffsschicht für externe Zugriffe.  
Home Assistant behält zusätzlich seine eigene Benutzerverwaltung.

---

## 7.3 Netzwerkstrategie

Geplant:
- VLANs
- getrennte IoT-Netze
- getrennte Kamera-Netze
- getrenntes Mediensegment
- zentrale Firewall-Regeln
- kontrollierte externe Zugriffe

---

## 7.4 Secrets-Management

- `secrets.yaml` für alle Tokens, API-Keys, Passwörter
- keine Secrets in Klartext in Automationen oder Dashboards
- regelmäßige Rotation kritischer Tokens
- API-Keys ausschließlich aus `secrets.yaml` referenzieren
- bei Bedarf späterer externer Secrets-Manager (Bitwarden / Vaultwarden)
- Backups der Konfiguration enthalten Secrets verschlüsselt

---

## 7.5 Multi-User & Family Access

### Manon: Apple Home als Primärinterface

Manon nutzt Apple Home als gewohnte Steuerung. JARVIS muss diese Welt sauber spiegeln.

Setup:
- HomeKit Bridge Integration in Home Assistant
- definierter Satz an Entitäten wird zu HomeKit gebridged
- Benennung der gebridged Devices bleibt konsistent mit JARVIS-Namenskonvention
- Änderungen am Bridge-Setup werden dokumentiert

Was gebridged wird:
- Licht (Hue über HA-Bridge zu HomeKit)
- Szenen
- ausgewählte Sensoren

Was nicht gebridged wird:
- interne Automationslogiken
- sicherheitskritische Aktoren
- Energiesteuerung

---

# 8. 💾 Backup & Disaster Recovery

---

## 8.1 Grundprinzipien

- regelmäßige automatische Backups
- redundante Speicherorte
- regelmäßige Restore-Tests
- Konfiguration als Code

---

## 8.2 Backup-Strategie

### Nächtlicher Clone auf Synology

- vollständiges Home-Assistant-Backup täglich nachts
- Ziel: Synology DS918+
- Synology Backup-Retention (TBD, Vorschlag: 7 daily + 4 weekly + 6 monthly)
- Benachrichtigung bei Backup-Fehlschlag

### Konfiguration als Git

- HA-Konfiguration als Git-Repository
- Versionierung aller YAML-Dateien
- Studio Code Server Add-on für Bearbeitung
- Secrets bleiben außerhalb des Repositories

### VM-Backups (Proxmox)

- vollständiges VM-Level-Backup (HAOS, JarvisBrain, Jarvis Services) über einen **Proxmox Backup Server (PBS)**
- PBS wird **später dezentral aufgesetzt und betrieben**
- bis dahin trägt der Kern-Restore über natives HA-Backup + Konfigurations-Git + HA-Green-Standby (8.3)
- bewusste Staffelung gemäß „Integrationskomplexität erst bei Bedarf"

### Offsite-Backup (optional)

- verschlüsselte Kopie auf externes Ziel (z. B. Backblaze B2 oder Synology C2)
- schützt vor lokalem Totalverlust

---

## 8.3 Restore-Strategie

- dokumentierter Wiederherstellungspfad
- Restore-Test mindestens halbjährlich
- Wiederherstellung auf separater Hardware testen, nicht produktiv
- Dokumentation des Recovery-Verfahrens in diesem Dokument oder im Wiki

### Wiederherstellungspfad „HA Green als Warm-Standby"

1. NUC-Ausfall erkannt (Monitoring, vgl. Kap. 9).
2. Home Assistant Green booten.
3. Letztes nächtliches HA-Backup von der Synology einspielen.
4. Kernsteuerung läuft wieder; JarvisBrain & Services folgen, sobald NUC ersetzt bzw. PBS-Restore verfügbar.

Damit ist der Single-Hardware-Punkt des NUC abgesichert, ohne den Resilienz-Grundsatz (0.3) zu verletzen.

---

## 8.4 Was wird gesichert?

| Komponente | Backup-Methode | Ziel |
|---|---|---|
| Home Assistant (HAOS-VM) | natives HA-Backup (täglich) | Synology |
| HA-Konfiguration | Git-Repository | Synology + extern |
| Datenbank (Recorder) | im HA-Backup enthalten | Synology |
| Grocy↔Mealie-Glue (HA `rest_command`) | in HA-Konfiguration / HA-Backup enthalten | Synology |
| n8n (Ausbaustufe, falls genutzt) | über Multiversum Service Stack gesichert | dezentral |
| Grocy | Datenbank-Export (SQLite) | Synology |
| Mealie | DB-/Daten-Export | Synology |
| Proxmox-VMs (HAOS, JarvisBrain, Services) | Proxmox Backup Server (geplant, dezentral) | dezentrales PBS-Ziel |
| UniFi Protect | UniFi-eigenes Backup | Synology |

---

# 9. 📊 Monitoring & Alerting

---

## 9.1 Zielsetzung

JARVIS überwacht sich selbst – nicht andersherum.

Folgendes muss früh erkannt werden:
- Sprachsatellit offline
- Automation läuft nicht mehr
- Backup fehlgeschlagen
- Gerät nicht mehr erreichbar
- ungewöhnlich hoher Stromverbrauch
- Netzwerkausfall
- Sensor liefert keine Werte mehr
- Proxmox-Host oder einzelne VM offline

---

## 9.2 Werkzeuge

| Werkzeug | Aufgabe |
|---|---|
| Uptime Kuma (auf Synology) | Verfügbarkeit von Diensten & Endpunkten |
| Watchman Integration | HA-interne Entity-Checks |
| HA System Monitor | Systemressourcen |
| NUT | USV-Status |
| Glances (optional) | Detail-Monitoring Synology / Proxmox |

---

## 9.3 Alert-Prioritäten

| Stufe | Beschreibung | Beispiel | Kanal |
|---|---|---|---|
| Kritisch | sofortige Reaktion notwendig | Stromausfall, HA down, Kamera-Ausfall | Push + ggf. SMS |
| Warnung | Reaktion innerhalb 24h | Backup fehlgeschlagen, Sensor offline | Push |
| Info | passive Kenntnisnahme | Update verfügbar, Statistik | Dashboard |

---

# 10. 🔔 Benachrichtigungsarchitektur

---

## 10.1 Zielsetzung

Zentrale, konsistente Benachrichtigungsstrategie über alle Subsysteme – kein Wildwuchs.

---

## 10.2 Kanäle

| Kanal | Verwendung |
|---|---|
| Home Assistant Companion App | primärer Push-Kanal |
| ntfy (selbst gehostet) | Fallback & generische Push-Empfänger |
| Sprachausgabe via Satelliten | situative Ansagen |
| Küchen-Dashboard | passive Anzeige |
| E-Mail (optional) | Logs, Zusammenfassungen |

---

## 10.3 Prioritätsstufen

- **Kritisch:** sofortiger Push + Sprachansage, ignoriert Stummschaltung
- **Warnung:** Push, kein Override
- **Info:** still im Dashboard, keine Push
- **Erinnerung:** kontextabhängig (z. B. Müll am Vorabend)

---

## 10.4 Regeln

- jede Benachrichtigung hat klar definierte Priorität
- keine doppelten Benachrichtigungen aus mehreren Subsystemen
- Benachrichtigungen für Manon laufen separat (über Apple Home oder eigener Push)
- nächtliche Stummphase außer für kritische Alerts

---

# 11. 📱 Dashboard-Konzept

---

## 11.1 Hauptdashboard

Anzeige:
- Uhrzeit
- Wetter
- Hausstatus
- Temperatur
- Kameras
- Energieübersicht
- Gartenstatus
- Wallbox
- Einkaufslisten
- Müllkalender
- Erinnerungen
- Medienstatus

---

## 11.2 Raum-Dashboards

Funktionen:
- Licht
- Temperatur
- Szenen
- Präsenz
- Mediensteuerung

---

## 11.3 Spezial-Dashboards

### Garage

Anzeige:
- BMW i3
- OpenWB
- Ladezustand
- Energieverbrauch

---

### Garten

Anzeige:
- Navimow
- Wetter
- Bewässerung
- Sensorik
- Gartenplaner

---

### Medien

Anzeige:
- Jellyfin
- Filmstatus
- Mediensteuerung
- Streamingübersicht
- Synology-Status

---

### Haushalt

Anzeige:
- Einkaufslisten
- Müllkalender
- Lagerverwaltung
- Erinnerungen
- Rezepte & Essensplanung (Mealie)

---

### System / Health

Anzeige:
- Backup-Status
- USV-Status
- Uptime Kuma Übersicht
- Sprachsatelliten-Status
- Proxmox-/VM-Status
- offene Warnungen

---

# 12. 🔌 Geräteintegration

---

## 12.1 Lichtsteuerung

### Philips Hue Bridge Pro

Verwendungszweck:
- zentrale Lichtsteuerung
- Szenen
- Automationen
- Präsenzsteuerung
- Dashboard-Integration
- Sprachsteuerung

Stand-alone-Verhalten:
- Hue Bridge funktioniert ohne JARVIS weiter
- Schalter und App bleiben nutzbar

---

## 12.2 Kameras

### UniFi Protect

Geplant:
- lokale Kameraverwaltung
- Home-Assistant-Integration
- Ereignissteuerung
- Dashboard-Integration
- spätere Einbindung in Sicherheits- und Präsenzlogiken

### Frigate (optional / ergänzend)

Für intelligente Objekterkennung:
- Person / Auto / Paket / Tier
- definierte Zonen
- Coral USB Accelerator empfohlen für schnelle Inferenz
- läuft auf Synology oder als VM/Container auf dem Proxmox-Host
- ergänzt UniFi Protect, ersetzt es nicht

---

## 12.3 Wallbox

### OpenWB

Geplant:
- Ladeverwaltung
- PV-Überschussladen
- Energiemanagement
- Verbrauchsanalysen
- Integration des BMW i3

Stand-alone-Verhalten:
- Default-Lademodus auch ohne JARVIS verfügbar

---

## 12.4 Garten

### D.U.M.E. (Segway Navimow i208)

Codename des Mähroboters:

**Offiziell:** Dynamic Unit for Maintenance & Ecology
**Inoffiziell:** Dümmste Unbemannte Mäh-Einheit

Geplant:
- Zonensteuerung
- Automationen
- Wetterintegration
- Statusüberwachung
- Gartenplaner

---

## 12.5 Heizung

### T-Arc (Thermal Arc) – *Hardware-Tendenz: Solvis Max mit Wärmepumpe*

Codename für das zukünftige Heizungssystem. Aktuelle Tendenz: **Solvis Max** als Heizzentrale in Kombination mit einer **Wärmepumpe** (final noch nicht beauftragt).

Geplante Funktionen:
- intelligente Heizungssteuerung
- Raumzonen
- Verbrauchsoptimierung
- Automatisierungen
- Energieintegration

Stand-alone-Verhalten:
- klassische Thermostat-Funktion muss erhalten bleiben
- Heizungsausfall durch JARVIS-Ausfall ist nicht akzeptabel

---

## 12.6 PV & Energiespeicher

### Arc-Reaktor – *Platzhalter, Hardware TBD*

Codename für das zukünftige PV- und Speichersystem. Konkrete Hardware noch nicht festgelegt.

Geplante Funktionen:
- PV-Management
- Energiespeicher
- Verbrauchssteuerung
- Energiemonitoring
- Smart-Grid-Logik

---

## 12.7 Medien

### Synology DS918+ / Jellyfin

Geplant:
- Medienstatus im Dashboard
- Steuerung von Jellyfin-Clients
- Szenen wie Filmabend
- Verbindung zu `cinema.multiversum.network`

---

## 12.8 Wetterstation

### Ecowitt Wetterstation

Geplant:
- lokale Wetterdaten
- Temperatur
- Luftfeuchtigkeit
- Wind
- Regenmessung
- Wetterlogik für Automationen

Verwendungszwecke:
- Gartenbewässerung
- Navimow-Steuerung
- Heizungslogik
- PV-Prognosen
- Wetterwarnungen

---

## 12.9 Gartenbewässerung

Geplant:
- zentrale Steuerung über Home Assistant
- 4 Magnetventile im Keller
- zonenbasierte Bewässerung
- wetterabhängige Steuerung
- spätere Automatisierung über Wetterstation

Geplante Zonen:
- Gemüsebeet
- Gartenstreifen
- Gewächshaus / Erweiterungen
- Reservezone

Stand-alone-Verhalten:
- bei JARVIS-Ausfall: Default-Zustand „aus"
- manueller Override am Ventil möglich

---

## 12.10 Smart Meter / Energiedaten

### Smarty (Luxemburg) via P1-Port

Geplant:
- Auslesung des Smarty-Zählers über P1-Port
- Lesegerät TBD (z. B. HomeWizard P1, Shelly Pro 3EM, P1 Monitor)
- Datenquelle für Energiemanagement, PV-Überschussladen und Lastmanagement

Funktionen:
- Live-Verbrauch
- Bezug & Einspeisung
- Tages-/Monats-/Jahresstatistiken
- Basis für intelligente Steuerung von OpenWB und späterem Energiespeicher

Wichtig:
- ohne diese Datenquelle ist sauberes PV-Überschussladen nicht möglich
- daher in Phase 2 vorgezogen

---

## 12.11 Präsenzerkennung

Geplant:
- mmWave-Sensoren pro Raum
- Beispiele: Aqara FP2, Apollo MSR-2 (Hardware TBD)

Vorteile gegenüber klassischer Bewegungssensorik:
- erkennt auch ruhende Personen
- Zonen pro Raum
- zuverlässiger als Bluetooth- oder WLAN-Tracking
- Grundlage für sinnvolle Raum-Automationen

Verwendungszwecke:
- Lichtautomationen
- Heizungssteuerung
- Anwesenheits-Szenen
- Sicherheits- und Energie-Logiken

---

## 12.12 Organisations- & Alltagssysteme

### Müllkalender

Verwendetes System:
- Waste Collection Schedule

Funktionen:
- automatische Erinnerungen
- Dashboard-Integration
- Sprachabfragen
- Push-Benachrichtigungen

---

### Gartenkalender

Verwendetes System:
- Home Assistant Calendar

Funktionen:
- Aussaatplanung
- Bewässerungsplanung
- saisonale Erinnerungen
- Gartenjournal
- Wetterintegration

---

### Lagerverwaltung & Einkauf

Verwendetes System:
- **Grocy** (self-hosted, Docker / LinuxServer.io, SQLite, REST-API, MIT)

Funktionen:
- Lebensmittel- & Bestandsverwaltung
- Haushaltsverbrauch & Mindestbestände
- Einkaufslisten
- Barcode-Unterstützung

Konfiguration:
- nicht benötigte Module (u. a. **Recipes**, Tasks, Batteries, Equipment, Calendar) über `FEATURE_FLAG_*` in `config.php` deaktiviert.
- SQLite ist für Single-Household der korrekte Backend — keine Migration auf MySQL/PostgreSQL.
- Betrieb in der **Jarvis-Services-VM**.

Mobile Nutzung:
- Smartphone-App für Lagerverwaltung
- Barcode-Scanner
- Einkaufslisten-Synchronisation

---

### Rezepte & Essensplanung

Verwendetes System:
- **Mealie** (self-hosted, API-fähig) — gewählt wegen schlanker, aufgeräumter UI (Usability für Manon).

Funktionen:
- Rezeptverwaltung & -import
- Essens-/Wochenplanung
- generierte Einkaufslisten
- Betrieb in der **Jarvis-Services-VM**

Rollenteilung:
- **Grocy** = Bestand + Einkauf (Recipes-Modul deaktiviert)
- **Mealie** = Rezepte + Essensplanung
- saubere Funktionstrennung statt Doppelpflege.

---

### Integration Grocy ↔ Mealie

- lose Kopplung über die REST-APIs beider Systeme; **Home Assistant** als Orchestrierungs- und Dashboard-Layer.
- **Startpunkt:** Free-Text-Sync über HA `rest_command` + Automation — HA pollt Mealies Einkaufsliste (REST-Sensor) und schiebt neue Posten als Freitext an die Grocy-API. Kein zusätzlicher Dienst im Stack, Logik bleibt in der HA-Konfiguration.
- **Ausbaustufe:** Bei wachsender Glue-Logik (Zutaten-Mapping, Dedup, Mengen) Auslagerung an **n8n** aus dem Multiversum Service Stack — kein eigener Runtime im JARVIS-Kern nötig.
- **Bewusst aufgeschoben:** vollständiges Zutaten-Mapping/Sync, bis reale Nutzungsmuster es rechtfertigen.
- Prinzip: „Stabilität vor Spielerei" + „erst lose, dann gezielt nachschärfen".

---

# 13. 📐 Konventionen & Namensschema

---

## 13.1 Zielsetzung

Mit wachsender Geräteanzahl entsteht ohne klare Konvention Wildwuchs. Frühe Festlegung spart spätere Migration.

---

## 13.2 Entitäts-Namensschema

Schema:
```
<raum>_<geraet>_<funktion>
```

Beispiele:
- `kueche_decke_licht`
- `lounge_couch_lampe`
- `garage_openwb_ladestand`
- `cafe_d_udange_dume_status`
- `haus_pv_einspeisung` (hausweite Entität)

---

## 13.3 Raumliste (verbindlich)

> **Kanonische Quelle:** Die vollständige Raum- und Hausstruktur ist in
> `Hausstruktur-Udange.json` (Schema 1.0) definiert. Diese Datei ist die
> maßgebliche Referenz inkl. Aliases, Adjazenzen und Notes; die folgenden
> Tabellen sind eine Lese-Übersicht und werden mit der JSON konsistent
> gehalten.

### Etagen

| Kürzel | Etage | Aliases |
|---|---|---|
| `ug` | Untergeschoss | Keller, UG, unten |
| `eg` | Erdgeschoss | EG, Parterre, Erdgeschoss |
| `og` | Obergeschoss | OG, oben, Obergeschoss |

### Übergeordnete Bereiche (Areas)

| ID | Bereich | Etage | Art |
|---|---|---|---|
| `innenbereich_untergeschoss` | Innenbereich Untergeschoss | UG | indoor |
| `aussenbereich_hinten` | Außenbereich hinten | UG | outdoor (hinter dem Haus) |
| `innenbereich_erdgeschoss` | Innenbereich Erdgeschoss | EG | indoor |
| `aussenbereich_vorne` | Außenbereich vorne | EG | outdoor (Eingangsseite) |
| `innenbereich_og` | Innenbereich Obergeschoss | OG | indoor |
| `aussenbereich_og` | Außenbereich Obergeschoss | OG | outdoor (Balkon) |

### Räume & Zonen — Untergeschoss

| ID | Name | Art |
|---|---|---|
| `garage` | Garage | room |
| `atelier` | Atelier | room |
| `waschkueche_vorratsraum` | Waschküche & Vorratsraum | room |
| `ruempelkammer` | Rümpelkammer | room |
| `ug_flur` | UG-Flur | room |
| `studio` | Studio | room |
| `terrasse` | Terrasse | zone (outdoor) |
| `treppe_terrasse` | Treppe Terrasse | zone (outdoor) |
| `cafe_d_udange` | Café d'Udange | zone (outdoor) |
| `gartenweg` | Gartenweg | zone (outdoor) |
| `ziergarten` | Ziergarten | zone (outdoor) |
| `nutzgarten` | Nutzgarten | zone (outdoor) |
| `garageneinfahrt` | Garageneinfahrt | zone (outdoor) |

### Räume & Zonen — Erdgeschoss

| ID | Name | Art |
|---|---|---|
| `eingang` | Eingang | room |
| `flur` | Flur | room |
| `kueche` | Küche | room |
| `esszimmer` | Esszimmer | room |
| `lounge` | Lounge (Split-Level) | room |
| `schlafzimmer` | Schlafzimmer | room |
| `duschzimmer` | Duschzimmer | room |
| `klo` | Klo | room |
| `treppe_obergeschoss` | Treppe Obergeschoss | room (stairs) |
| `treppe_untergeschoss` | Treppe Untergeschoss | room (stairs) |
| `zugang` | Zugang | zone (outdoor) |
| `parkplatz` | Parkplatz | zone (outdoor) |
| `spielwiese` | Spielwiese | zone (outdoor) |

### Räume & Zonen — Obergeschoss

| ID | Name | Art |
|---|---|---|
| `mezzanine` | Mezzanine | room (split_level) |
| `flur_og` | Flur OG | room |
| `speicher` | Speicher | room |
| `badezimmer` | Badezimmer | room |
| `schlafzimmer_c` | Schlafzimmer C | room |
| `balkon` | Balkon | zone (outdoor) |

### Hausweite Entitäten

| Kürzel | Verwendung |
|---|---|
| `haus` | hausweite Entitäten (z. B. `haus_pv_einspeisung`, `haus_alarm_status`) |

### Strukturelle Besonderheiten

- **Eine obere Treppe:** `treppe_obergeschoss` ist die einzige Treppe vom EG-Flur zur Mezzanine im OG (zusammengeführt aus EG- und OG-Modell).
- **Hintere Außenzonen kanonisch im UG:** Terrasse, Gartenweg, Café d'Udange, Zier-/Nutzgarten und Garageneinfahrt liegen alle unter `aussenbereich_hinten` (UG), auch wenn die Terrasse physisch auf EG-Ebene ist. Dadurch ist der hintere Außenbereich nur einmal definiert.
- **Split-Level:** `lounge` (EG) und `mezzanine` (OG) sind als `level_type: split_level` markiert und werden trotzdem fest einer Etage zugeordnet.
- **Lounge ≠ Wohnzimmer:** Die JSON kennt kein „Wohnzimmer"; die Lounge ist der Lounge-/Sitzbereich auf der EG-Split-Level-Ebene über dem Esszimmer.

---

## 13.4 Funk-Stack

Koordinatoren (entschieden, siehe 3.7):
- **Hue Bridge Pro** – dedizierter Hue-Zigbee-Stack
- **Connect ZBT-2** – allgemeines Zigbee/Thread/Matter
- **Connect ZWA-2** – Z-Wave 800 (EU)

Software-Integrationen:
- **Z-Wave:** Z-Wave JS (über ZWA-2)
- **Zigbee:** Integration über ZBT-2 — Entscheidung ZHA vs. Zigbee2MQTT noch offen:
  - **ZHA:** offiziell, einfache Einrichtung, weniger Device-Support
  - **Zigbee2MQTT:** größere Device-Kompatibilität, aktivere Community, mehr Konfigurierbarkeit
  - Empfehlung: Zigbee2MQTT wegen Gerätebreite und Zukunftssicherheit.

---

## 13.5 Dokumentationsprinzipien

- jede neue Integration wird hier oder in Anhang dokumentiert
- Codenamen (D.U.M.E., T-Arc, Arc-Reaktor) bleiben als feste Bezeichner erhalten; bei T-Arc / Arc-Reaktor bis zur finalen Hardware-Entscheidung
- Änderungen an der Netzwerksegmentierung werden versioniert

---

# 14. 🚀 Planung

---

## Phase 1 – Grundsystem

- Intel NUC mit Proxmox VE (Virtualisierungsbasis)
- 3-VM-Setup: HAOS / JarvisBrain / Jarvis Services
- Home Assistant OS (HAOS-VM)
- Home Assistant Voice Preview Edition
- Grundkonfiguration
- Sprachsteuerung
- Hue Integration
- Müllkalender
- Einkaufslisten
- Lagerverwaltung (Grocy)
- Rezepte/Essensplanung (Mealie)
- nächtliches Backup auf Synology
- Git-Repository für Konfiguration
- Home Assistant Green als Cold/Warm-Standby konfigurieren
- USV für Kernsysteme
- Namenskonvention festlegen

Wichtig:
- Manon behält Apple Home zur Steuerung

---

## Phase 2 – Alltag & Infrastruktur

- Küchen-Tablet
- Dashboard-Aufbau
- Jellyfin-Integration
- Synology-Status
- Netzwerksegmentierung
- Remote-Zugriff über Authentik / Tailscale
- erste Haushaltsroutinen
- Sprachsatelliten
- Smarty / P1-Anbindung (vorgezogen wegen Energiebasis)
- Uptime Kuma & Monitoring-Grundgerüst
- Benachrichtigungsarchitektur aufgesetzt
- mmWave-Präsenzsensoren in Hauptzonen

---

## Phase 3 – Garten & Garage

- OpenWB
- BMW i3 Integration
- Segway Navimow i208 / E208
- Gartenplaner
- Wetterlogik
- Kamera-System über UniFi Protect
- optional: Frigate auf Synology / Proxmox
- Gartenbewässerung
- Ecowitt Wetterstation
- Hermes Agent (JarvisBrain) mit DeepSeek-Backend (Übergang)

---

## Phase 4 – Energie & Heizung

- T-Arc / Thermal Arc (Hardware-Entscheidung)
- Arc-Reaktor (Hardware-Entscheidung)
- PV-Management
- Energiespeicher
- Heizungsintegration
- Energiemanagement auf Basis Smarty + PV

---

## Phase 5 – Erweiterte KI-Assistenz

- Hermes Agent als zentraler Conversation Agent etabliert
- Evaluation: Umschwenk LLM-Backend von DeepSeek (Infomaniak) auf lokales Ollama
- KI-gestützte Routinen
- Energieoptimierung
- Alltagsempfehlungen
- erweiterte Sprachdialoge

---

# 15. 🔬 Forschung

---

## 15.1 Autonome Überwachungsdrohnen (Langfristig)

Indoor & Outdoor

Mögliche spätere Integration:
- Kontrollflüge
- Grundstücksüberwachung
- Kamera-Streams
- ereignisgesteuerte Einsätze
- Integration in Home Assistant

Status:
- Konzeptphase

### Rechtliche Anmerkung

In der EU gilt die Drohnenverordnung 2019/947. Autonome Flüge zur Grundstücksüberwachung sind stark reglementiert:

- Außenflüge: in der Regel Sichtkontakt-Pflicht, BVLOS nur mit Sondergenehmigung
- Datenschutz: Aufnahme öffentlicher / nachbarlicher Bereiche problematisch (DSGVO)
- Indoor-Flüge: rechtlich entspannter, technisch anspruchsvoller

Konzept bleibt deshalb langfristig und vorbehaltlich rechtlicher Klärung.

---

# 📎 Referenzen

- Multiversum Systemarchitektur
- Infrastruktur-Dokumentation
- Sicherheitsrichtlinien
- Netzwerk-Dokumentation
- Home-Assistant-Dokumentation
- Proxmox-VE-Dokumentation
- Hermes Agent (Nous Research)
- Home Assistant Connect ZBT-2 / ZWA-2 (Nabu Casa)
- Z-Wave JS Dokumentation
- Jellyfin-Dokumentation
- UniFi-Dokumentation
- EU-Drohnenverordnung 2019/947

---

# 📝 Changelog

| Version | Änderungen |
|---|---|
| 1.4 | Foundation Draft (Ausgangsbasis) |
| 1.5 | + Kapitel 0 (Designprinzipien & Nicht-Ziele) + Resilienz-Prinzip + USV (3.6) + Secrets-Management (7.4) + Multi-User & Apple Home (7.5) + Backup & Disaster Recovery (Kap. 8) + Monitoring & Alerting (Kap. 9) + Benachrichtigungsarchitektur (Kap. 10) + System/Health-Dashboard (11.3) + Conversation Agent (5.5) + Frigate-Option (12.2) + Smart Meter / Smarty (12.10) + mmWave-Präsenzerkennung (12.11) + Konventionen & Namensschema (Kap. 13) + Rechtliche Anmerkung Drohnen (15.1) + T-Arc / Arc-Reaktor als Platzhalter markiert |
| 1.6 | Konsolidierung auf Proxmox VE (Intel NUC) als Kernplattform mit 3-VM-Setup (HAOS / JarvisBrain / Jarvis Services) (3.1, 3.3, 4.1) · HA Green → Cold/Warm-Standby inkl. Restore-Pfad (0.3, 3.1, 8.3) · Hermes Agent (Nous Research) als JarvisBrain mit austauschbarem LLM-Backend (5.5, 6.2, 14) · Mealie statt Tandoor für Rezepte/Essensplanung + überarbeitete Rollenteilung Grocy/Mealie (11.3, 12.12, 14) · Grocy↔Mealie-Sync über HA `rest_command` (Start), n8n als Ausbaustufe (12.12) · VM-Backups über künftiges dezentrales PBS, interim HA-Backup + Standby (8.2, 8.4) |
| 1.7 | Funkadapter & Koordinatoren ergänzt: Hue Bridge Pro + offizielle HA Connect ZBT-2 (Zigbee/Thread) & ZWA-2 (Z-Wave 800 EU) (3.7, 4.1, 13.4) · Navimow als D.U.M.E. (Dynamic Unit for Maintenance & Ecology) benannt (12.4, 13.5) · T-Arc Hardware-Tendenz: Solvis Max mit Wärmepumpe (12.5) · Raumliste auf kanonische Quelle `Hausstruktur-Udange.json` umgestellt (13.3) · Referenzen ergänzt |
| 1.8 | Raumliste (13.3) durch tatsächliche Struktur aus `Hausstruktur-Udange.json` (Schema 1.0) ersetzt: Etagen + übergeordnete Areas + vollständige Räume/Zonen pro Etage + hausweite Entitäten + strukturelle Besonderheiten (eine obere Treppe, hintere Außenzonen kanonisch im UG, Split-Level Lounge/Mezzanine) · Beispiele in 13.2 an reale IDs angepasst (Lounge statt Wohnzimmer, `cafe_d_udange_dume_status` statt generisches Garten-Beispiel) |

---

# 🧠 Status

Version: 1.8  
Typ: Foundation Draft  
Status: aktiv in Entwicklung