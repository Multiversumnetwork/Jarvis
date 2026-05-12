# Apple Home und HomeKit Bridge

## Ziel

Manon soll weiterhin Apple Home verwenden können.

Home Assistant wird die technische Zentrale, Apple Home bleibt für Manon die vertraute Oberfläche.

## Architektur

```text
Geräte
    ↓
Home Assistant
    ↓
HomeKit Bridge
    ↓
Apple Home
    ↓
Manon
```

## Grundsatz

Migration ohne Bruch.

Keine harte Umstellung, sondern Parallelbetrieb.

## Vorgehen

### Schritt 1

Home Assistant parallel aufsetzen.

### Schritt 2

Geräte inventarisieren.

### Schritt 3

Erste Geräte in Home Assistant integrieren.

### Schritt 4

HomeKit Bridge aktivieren.

### Schritt 5

Ausgewählte Entitäten nach Apple Home exportieren.

### Schritt 6

Apple-Home-Szenen schrittweise nach Home Assistant überführen.

## Für Manon sichtbar

Nur stabile, alltagstaugliche Entitäten:

- Licht
- einfache Szenen
- Heizung
- Steckdosen
- ggf. Rollläden
- keine komplexen Diagnose-Sensoren

## Nicht exportieren

- Debug-Sensoren
- interne Helfer
- technische Automationen
- experimentelle Entitäten
- Kameras nur nach bewusster Entscheidung

## Zielzustand

Manon kann weiter Apple Home nutzen.

Marc und JARVIS arbeiten über Home Assistant.
