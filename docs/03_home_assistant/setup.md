# Home Assistant Setup

## Ziel

Home Assistant ist das technische Nervensystem von JARVIS Home.

## Installation

Startplattform:

- Home Assistant OS
- offizielle Nabu-Casa-Hardware
- primär Home Assistant Green

## Grundkonfiguration

Wichtige Punkte:

- feste IP-Adresse
- saubere Namenskonvention für Entitäten
- Backup-Strategie ab Tag 1
- getrennte Bereiche/Räume
- Geräte sauber dokumentieren
- keine wilden Automationen ohne Benennung

## Namenskonvention

### Räume

Beispiele:

```text
kueche
wohnzimmer
schlafzimmer
bad
buero
waschkueche
garage
garten
```

### Entitäten

Beispiele:

```text
light.wohnzimmer_decke
sensor.bad_temperatur
switch.kaffeemaschine
binary_sensor.garage_tor
climate.bad_heizung
```

## Bereiche

Alle Geräte werden direkt Bereichen zugeordnet.

## Backups

Pflicht:

- Backup vor größeren Änderungen
- Backup vor Add-on-Installationen
- Backup vor Migrationen
- Backup vor Automationsumbauten

## Add-ons und Integrationen

Geplant:

- HomeKit Bridge
- Grocy
- OpenRouter
- UniFi Protect
- Wetterintegration
- Kalenderintegration
- MQTT optional
- ESPHome
- File Editor / Studio Code Server optional

## Dokumentationspflicht

Jede wichtige Integration bekommt eine eigene Markdown-Datei im Repo.
