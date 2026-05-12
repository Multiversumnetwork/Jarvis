# Systemarchitektur

## Überblick

```text
JARVIS Home
├── Home Assistant OS
├── Voice Assistant
├── OpenRouter LLM
├── HomeKit Bridge
├── Grocy
├── Kalender- und Wetterlogik
├── UniFi Protect
├── Dashboards
└── spätere Drohnenintegration
```

## Rollenverteilung

### JARVIS

- Haussteuerung
- Sensorik
- Szenen
- Statusmeldungen
- Voice Interface
- Vorräte
- Gartenkalender
- Kameras

### Pepper

- Marc
- Projekte
- Architektur
- Dokumentation
- kreative und technische Planung

### Raymond

- Manon
- Unterstützung im Alltag
- Recherche und Erklärungen
- technische Hilfe für Manon

### SINA

- Analyse
- Produktionsprozesse
- Monitoring
- Workflow-Dokumentation

## Technischer Kern

```text
Home Assistant OS
    ↓
Integrationen / Geräte / Sensoren
    ↓
Assist Pipeline
    ↓
OpenRouter LLM
    ↓
JARVIS Antwort / Aktion
```

## GitOps-Prinzip

Konfigurationen und Dokumentation sollen versioniert werden.

```text
Vorschlag → Dateiänderung → Review → Commit → Push → Deployment
```

Direkte, nicht dokumentierte Änderungen in Home Assistant sollen möglichst vermieden oder nachträglich ins Repo übernommen werden.

## Kontrollierte Selbstverbesserung

JARVIS darf:

- Muster erkennen
- Probleme analysieren
- Optimierungen vorschlagen
- neue Szenen vorschlagen
- YAML vorbereiten
- Change Requests erzeugen

JARVIS darf nicht ohne Freigabe:

- sicherheitskritische Systeme ändern
- Heizlogik grundlegend umbauen
- Tür-/Alarm-/Kamera-Logik ändern
- Benutzer/Rechte ändern
- Netzwerk oder Backups verändern
