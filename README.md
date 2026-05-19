# JARVIS Home Assistant

JARVIS ist die intelligente Haussteuerung des `Multiversum.network`.

Dieses Repository dokumentiert Aufbau, Architektur, Hardware, Automationen und Betriebsregeln für das JARVIS-Home-System.

## Leitprinzip

> **JARVIS darf intelligent sein, aber nicht chaotisch.**

Änderungen laufen kontrolliert ab:

```text
Beobachten → Analysieren → Vorschlagen → Test/Staging → Freigabe → Umsetzung → Dokumentation → Rollback (falls nötig)
```

## Projektdefinition

**Aktueller Zustand:** Apple Home steuert einige Lampen sowie Heizkörper. Es existieren nur zwei bis drei Automatisierungen.

**Zielzustand:** JARVIS / Home Assistant ersetzt Apple Home vollständig. Apple Home bleibt nur für einen Nutzer als Frontend erhalten.

### Ziele

| # | Ziel | Status |
|---|------|--------|
| 1 | Steuerung aller aktuellen Lampen und Aktoren | `[ ]` |
| 2 | Integration einer KI (DeepSeek über Infomaniak, nicht lokal) | `[ ]` |
| 3 | Integration der Persona und deutscher JARVIS-Stimme | `[ ]` |
| 4 | Lagerverwaltung (Lebensmittelbestände) mit altem Handy als Barcode-Scanner | `[ ]` |
| 5 | Gartenkalender (Nutzgarten und Ziergarten) | `[ ]` |
| 6 | Küchen-Dashboard auf altem Android-Tablet | `[ ]` |
| 7 | Smart Mirror im Badezimmer (Raspberry Pi + MagicMirror²) | `[ ]` |

## Projektphasen

| Phase | Inhalt | Status |
|-------|--------|--------|
| 0 – Basis | Home Assistant Green, Voice Preview Edition, HACS, Netzwerk, Backup-Strategie | `[WIP]` |
| 1 – Migration | Lampen und Aktoren von Apple Home zu Home Assistant überführen | `[ ]` |
| 2 – Sprache & KI | DeepSeek (Infomaniak) anbinden, Persona-Stimme integrieren | `[ ]` |
| 3 – Vorräte | Grocy-Installation, Barcode-Scanner (altes Handy) | `[ ]` |
| 4 – Garten | Local Calendar, Pflegerhythmen, optional Plant-Sensorik | `[ ]` |
| 5 – Küchen-Dashboard | Android-Tablet mit Fully Kiosk Browser | `[ ]` |
| 6 – Smart Mirror | Raspberry Pi mit MagicMirror² im Badezimmer | `[ ]` |
| 7 – Erweitert | Kameras, Drohnen, Security | `[ ]` |

## Persona

JARVIS ist mehr als eine Stimme. Die vollständige Persona – Identität, Anrede, Tonalität, Beispieldialoge und Grenzen – ist im separaten Dokument [`PERSONA.md`](./PERSONA.md) definiert.

**Kurzfassung:**
- Förmlich, „Sir" und „Madam"
- Trocken-britischer Humor, sparsam eingesetzt
- Diskret, präzise, niemals hektisch
- Wake Word: `JARVIS`

## Rollen im Multiversum

- **JARVIS** – Haussteuerung, Sensorik, Szenen, Sprache, Status, Kameras, Vorräte
- **PEPPER** – persönlicher Assistent für Marc: Projekte, Architektur, Kreativität
- **RAYMOND** – persönlicher Assistent für Manon
- **SINA** – Analyse, Produktion, Monitoring und Workflow-Dokumentation
- **Home Assistant** – technisches Nervensystem des Hauses
- **n8n** – Workflow- und Automationsmotor

> Die Personas KAREN und NORA sind bewusst nicht aufgeführt, da sie nicht mit der Haussteuerung interagieren.

## Software-Stack

### Basis

- **Home Assistant OS** auf Home Assistant Green
- **HACS** (Home Assistant Community Store) als Erweiterungs-Layer für Custom Integrations, Lovelace-Cards und Themes
- **n8n** für komplexe Workflows außerhalb der HA-Automationen

### HACS – Geplante Integrationen und Cards

| Komponente | Zweck |
|------------|-------|
| `mushroom-cards` | UI-Cards für Dashboards |
| `bubble-card` | Touch-optimierte Cards (Küchen-Tablet) |
| `card-mod` | Styling-Anpassungen |
| `grocy` (Custom Integration) | Anbindung der Lagerverwaltung |
| `openplantbook` | Pflanzendatenbank für Gartenmodul |
| `browser_mod` | Steuerung des Tablet-Browsers von HA aus |
| `auto-entities` | Dynamische Card-Inhalte |
| `fully-kiosk-browser` | HA-Integration für das Küchen-Tablet |

### Lagerverwaltung – **Grocy**

- Open-Source, selbst gehostet (Docker-Container neben HA oder als HA-Add-on)
- Barcode-Scanning über das alte Android-Handy mit der App **Grocy Barcode Buddy** oder direkt über die Companion-App
- Funktionen: Bestände, Mindesthaltbarkeit, Einkaufsliste, Verbrauchsstatistik, Rezepte
- HA-Integration via HACS – Bestände werden als Entitäten verfügbar
- Optional: Anbindung an `Bring!` für die Einkaufsliste

### Gartenkalender

- **Home Assistant Local Calendar** (built-in) für Aufgaben: Aussaat, Düngen, Gießen, Schneiden, Ernte
- **OpenPlantbook** (HACS) als Pflanzendatenbank
- Optional **Plant Integration** (built-in) für Bodenfeuchte- und Lichtsensoren, falls Sensorik nachgerüstet wird
- Trennung von Nutzgarten und Ziergarten über zwei Kalender-Entitäten

### Sprachsteuerung

- **Voice Preview Edition** als Hardware
- **Wyoming Protocol** für lokale STT/TTS
- **Piper** für lokale deutsche TTS-Stimme (Fallback)
- **DeepSeek via Infomaniak** für KI-Anfragen
- Persona-Prompt versioniert in `04_voice_llm/`

### Küchen-Dashboard

- **Altes Android-Tablet** an der Küchenwand montiert
- **Fully Kiosk Browser** (kostenpflichtig, einmalig ca. 7 €) für Kiosk-Modus, Bewegungserkennung, Screen-Wake
- Dedizierte Dashboard-View in HA, touch-optimiert mit großen Buttons
- Inhalte: Wetter, Vorräte, Einkaufsliste, Gartenaufgaben, Heizung, Szenen, Anwesenheit
- Steuerung des Tablets durch HA via `browser_mod` und Fully-Kiosk-HACS-Integration (Screen ein/aus, Lautstärke, View-Wechsel)
- Wake bei Bewegung über PIR-Sensor (Kamera-basierte Erkennung ist ab Android 14 eingeschränkt)

### Smart Mirror Badezimmer

- **Raspberry Pi** (4 oder 5) hinter einem Spionspiegel mit Monitor
- **MagicMirror²** als Software-Plattform (Open Source, Node.js-basiert, modular)
- Display dauerhaft an, dimmt nachts ab
- Touch optional – Bedienung primär passiv (anschauen) und per Sprache über JARVIS
- Geplante MagicMirror²-Module:

| Modul | Funktion |
|-------|----------|
| `default/clock` | Uhrzeit und Datum |
| `default/weather` | Wetter und Vorhersage |
| `default/calendar` | Termine aus Local Calendar |
| `MMM-HomeAssistant` oder `MMM-Homeassistant-SwitchPlatform` | HA-Entitäten anzeigen (Heizung, Wetter, Anwesenheit) |
| `MMM-MQTT` | Bidirektionale Kopplung mit HA via MQTT |
| `MMM-Remote-Control` | Module von HA aus steuern |
| `MMM-Carousel` | Module rotieren (Morgens andere Inhalte als abends) |
| `MMM-Compliments` | Tageszeit-abhängige Begrüßung im JARVIS-Stil |

- Inhalte morgens: Uhrzeit, Wetter, Termine, Müllabfuhr, Ausgangsstatus (Fenster, Heizung)
- Inhalte abends: Uhrzeit, Wetter morgen, ruhigere Ansicht, gedimmt
- Auto-Modus-Wechsel über HA-Automation (Tageszeit + Anwesenheit)

## Nicht-funktionale Anforderungen

- **Privacy:** Sensordaten und persönliche Inhalte bleiben primär lokal. Nur explizit freigegebene Anfragen verlassen das Netzwerk in Richtung DeepSeek/Infomaniak.
- **Verfügbarkeit:** Fällt DeepSeek oder Infomaniak aus, bleibt die Kernsteuerung (Lampen, Heizung, Szenen) lokal funktionsfähig. Sprach- und KI-Features dürfen degradieren, Basisfunktionen nicht.
- **Lokale Fallbacks:** Kritische Automatisierungen laufen ohne Cloud-Abhängigkeit.

## Datenfluss (Übersicht)

```text
Apple Home (Frontend, 1 Nutzer)
        ↓
   Home Assistant  ←→  n8n (Automation)
        ↕                ↕
   Sensoren            Grocy (Lagerverwaltung)
   Aktoren             Local Calendar (Garten)
   Kameras             Browser Mod (Küchen-Tablet)
                       MQTT  ←→  MagicMirror² (Badezimmer)
        ↓
   DeepSeek via Infomaniak  (nur für Sprach- & KI-Anfragen)
```

Details siehe `01_architecture/`.

## Backup und Recovery

- Tägliche Snapshots des Home-Assistant-Systems (Konfiguration und Datenbank)
- Off-Site-Sicherung auf separatem Speicher
- Quartalsweiser Restore-Test, dokumentiert in `13_changelog/`
- Grocy-Datenbank separat gesichert (eigener Container)
- MagicMirror²-Konfiguration (`config.js` und Custom-CSS) in Git versioniert

## Hardware (Kurzübersicht)

- **Home Assistant Green** als zentrale Steuereinheit
- **Voice Preview Edition** für Sprachsteuerung
- **Zigbee/Matter** als primäre Funkstandards
- **Altes Android-Smartphone** als Barcode-Scanner für die Lagerverwaltung
- **Altes Android-Tablet** als Küchen-Dashboard (wandmontiert)
- **Raspberry Pi (4 oder 5)** mit Monitor und Spionspiegel als Smart Mirror im Badezimmer

Vollständige Liste: `02_hardware/`.

## Dokumentstruktur

```text
docs/
├── 00_project/        Projektübersicht, Vision, Ziele, PERSONA.md
├── 01_architecture/   Systemarchitektur und Datenflüsse
├── 02_hardware/       Geräte, Sensoren, Netzwerk, Tablet, Scanner-Handy, Raspberry Pi
├── 03_home_assistant/ Konfiguration, HACS, Integrationen
├── 04_voice_llm/      Sprachsteuerung, DeepSeek, Persona-Prompt
├── 05_apple_home/     Verbleibendes Apple-Home-Frontend
├── 06_scenes/         Szenen und Automationen
├── 07_inventory/      Grocy-Konfiguration und Barcode-Scanner
├── 08_garden/         Garten- und Pflegekalender
├── 09_cameras/        Kamerasystem (Phase 7)
├── 10_drones/         Drohnenintegration (Phase 7)
├── 11_security/       Sicherheitskonzept (Phase 7)
├── 12_smart_mirror/   MagicMirror² – Konfiguration, Module, Hardware-Aufbau
└── 13_changelog/      Änderungsprotokoll und Restore-Logs
```
