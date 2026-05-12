# Drohnenkonzept

## Ziel

Langfristig sollen eine Indoor- und eine Outdoor-Drohne in JARVIS Home integriert werden.

## Status

Dieses Thema ist ein späteres Subprojekt.

Keine Drohnenintegration in Phase 1.

## Grundsatz

Drohnen sind sicherheitskritisch.

JARVIS darf Drohnen nicht frei und autonom steuern, solange keine klaren Sicherheitsregeln, Zonen und Freigaben definiert sind.

## Mögliche Einsatzfälle

### Indoor

- Kontrollflug bei Abwesenheit
- Geräuschquelle prüfen
- Raumstatus visuell prüfen
- kein Flug in sensiblen Bereichen ohne Freigabe

### Outdoor

- Garten prüfen
- Grundstück kontrollieren
- Sturmschäden prüfen
- Kameraergänzung

## Sicherheitsregeln

Pflicht:

- manuelle Freigabe
- definierte Flugzonen
- No-Go-Zonen
- Not-Stopp
- Hinderniserkennung
- Protokollierung
- keine Flüge bei Menschen/Tieren im Gefahrenbereich
- keine Flüge bei schlechtem Wetter outdoor

## Architekturidee

```text
JARVIS
→ Vorschlag / Anfrage
→ Marc bestätigt
→ Drohnencontroller
→ Flug
→ Status / Ergebnis
```

## Kein Ziel

JARVIS soll keine Spielzeugdrohne zufällig durchs Haus jagen.

Drohnen werden als kontrollierte Sensorplattform verstanden.
