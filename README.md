# 🌌 JARVIS
# Haussteuerung & Smart-Home-System des Multiversums
## Version 1.5 – Foundation Draft

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

### Home Assistant Green

Verwendungszweck:
- Hauptsystem für Home Assistant
- Geräteverwaltung
- Automatisierung
- Dashboard-System
- Sprachintegration
- Smart-Home-Steuerung

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

Geplante Systeme:
- UniFi Netzwerk-Infrastruktur
- UniFi Protect
- VLAN-Struktur
- lokaler Proxmox-Server

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
- Home Assistant Green
- FritzBox
- UniFi-Core-Switch
- ggf. Proxmox-Server
- ggf. Synology DS918+

Funktionen:
- Überbrückung kurzer Stromausfälle
- sauberes Herunterfahren bei längerem Ausfall
- NUT-Integration in Home Assistant für Status & Automationen
- Benachrichtigung bei Netzausfall

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
JARVIS (Home Assistant)
   │
├── Zigbee / Matter
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

Über reine Intent-Matches hinaus benötigt Assist für natürlichsprachige Steuerung einen Conversation Agent.

Optionen:

| Option | Vorteile | Nachteile |
|---|---|---|
| Home Assistant Assist (Intent-basiert) | vollständig lokal, keine Abhängigkeit | nur definierte Phrasen |
| DeepSeek via Infomaniak AI | leistungsfähig, datenschutzkonformer Anbieter (EU/CH) | Cloud-Abhängigkeit |
| Ollama auf Proxmox (z. B. Llama 3.1 8B) | lokal, datenschutzfreundlich, unabhängig | benötigt geeignete Hardware (GPU empfohlen) |

Empfehlung:
- Phase 1–2: Intent-basiert
- Phase 3: DeepSeek als Übergang
- Phase 4+: Ollama auf Proxmox als finales Ziel

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
- DeepSeek über Infomaniak AI
- API-basierte KI-Integration
- lokale Sprachverarbeitung
- KI-gestützte Assistenzfunktionen
- optional: lokales LLM auf Proxmox (Ollama)

Nicht geplant:
- lokales LLM auf dem Home Assistant Green
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

### Offsite-Backup (optional)

- verschlüsselte Kopie auf externes Ziel (z. B. Backblaze B2 oder Synology C2)
- schützt vor lokalem Totalverlust

---

## 8.3 Restore-Strategie

- dokumentierter Wiederherstellungspfad
- Restore-Test mindestens halbjährlich
- Wiederherstellung auf separater Hardware testen, nicht produktiv
- Dokumentation des Recovery-Verfahrens in diesem Dokument oder im Wiki

---

## 8.4 Was wird gesichert?

| Komponente | Backup-Methode | Ziel |
|---|---|---|
| Home Assistant | natives HA-Backup (täglich) | Synology |
| HA-Konfiguration | Git-Repository | Synology + extern |
| Datenbank (Recorder) | im HA-Backup enthalten | Synology |
| Node-RED Flows (falls genutzt) | im HA-Backup enthalten | Synology |
| UniFi Protect | UniFi-eigenes Backup | Synology |
| Grocy | Datenbank-Export | Synology |

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
- spätere Kochrezepte

---

### System / Health

Anzeige:
- Backup-Status
- USV-Status
- Uptime Kuma Übersicht
- Sprachsatelliten-Status
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
- läuft auf Synology oder Proxmox in Docker
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

### Segway Navimow i208

Geplant:
- Zonensteuerung
- Automationen
- Wetterintegration
- Statusüberwachung
- Gartenplaner

---

## 12.5 Heizung

### T-Arc (Thermal Arc) – *Platzhalter, Hardware TBD*

Codename für das zukünftige Heizungssystem. Konkrete Hardware noch nicht festgelegt.

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

### Lagerverwaltung

Verwendetes System:
- Grocy

Funktionen:
- Lebensmittelverwaltung
- Haushaltsverbrauch
- Einkaufslisten
- Barcode-Unterstützung
- Mindestbestände
- Rezeptverwaltung

Mobile Nutzung:
- Smartphone-App für Lagerverwaltung
- Barcode-Scanner
- Einkaufslisten-Synchronisation

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
- `wohnzimmer_couch_lampe`
- `garage_openwb_ladestand`
- `garten_navimow_status`

---

## 13.3 Raumliste (verbindlich)

| Kürzel | Raum |
|---|---|
| kueche | Küche |
| wohnzimmer | Wohnzimmer / Lounge |
| schlafzimmer | Schlafzimmer |
| bad | Badezimmer |
| flur | Flur / Eingang |
| keller | Keller |
| garage | Garage |
| studio | Studio |
| garten | Garten |
| haus | hausweite Entitäten |

---

## 13.4 Zigbee-Stack

Entscheidung TBD:
- **ZHA:** offiziell, einfache Einrichtung, weniger Device-Support
- **Zigbee2MQTT:** größere Device-Kompatibilität, aktivere Community, mehr Konfigurierbarkeit

Empfehlung: Zigbee2MQTT wegen Gerätebreite und Zukunftssicherheit.

---

## 13.5 Dokumentationsprinzipien

- jede neue Integration wird hier oder in Anhang dokumentiert
- Codenamen (T-Arc, Arc-Reaktor) bleiben bis zur Hardware-Entscheidung als Platzhalter erhalten
- Änderungen an der Netzwerksegmentierung werden versioniert

---

# 14. 🚀 Planung

---

## Phase 1 – Grundsystem

- Home Assistant Green
- Home Assistant Voice Preview Edition
- Grundkonfiguration
- Sprachsteuerung
- Hue Integration
- Müllkalender
- Einkaufslisten
- Lagerverwaltung
- Rezeptdatenbank
- nächtliches Backup auf Synology
- Git-Repository für Konfiguration
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

- DeepSeek über Infomaniak AI als Conversation Agent
- KI-gestützte Routinen
- Energieoptimierung
- Alltagsempfehlungen
- erweiterte Sprachdialoge
- Evaluation: lokales LLM auf Proxmox (Ollama) als Ersatz für Cloud-Agent

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
- Jellyfin-Dokumentation
- UniFi-Dokumentation
- EU-Drohnenverordnung 2019/947

---
