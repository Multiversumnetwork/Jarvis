# Projektübersicht – JARVIS Home

## Ziel

JARVIS wird die zentrale intelligente Haussteuerung.

Er soll nicht Pepper ersetzen und nicht als allgemeiner persönlicher Assistent auftreten. JARVIS ist die Stimme und Steuerlogik des Hauses.

## Kernfunktionen

JARVIS soll übernehmen:

- Lichtsteuerung
- Heizungs- und Komfortszenen
- Sprachsteuerung
- Statusmeldungen
- Hausroutinen
- Wetterbasierte Planung
- Gartenkalender
- Vorratsverwaltung
- Einkaufsliste
- Kamera- und Sicherheitsintegration
- spätere Drohnenintegration
- Dashboard-Anzeige auf Tablets und Satelliten

## Grundsatz

JARVIS darf nicht unkontrolliert produktive Systeme umbauen.

Jede strukturelle Änderung wird dokumentiert, versioniert und kann rückgängig gemacht werden.

## Projektphasen

### Phase 1 – Basis

- Home Assistant OS auf offizieller Nabu-Casa-Hardware
- Home Assistant Voice Preview Edition
- OpenRouter als LLM-Anbindung
- erste Sprache und erste Szenen

### Phase 2 – Apple Home Parallelbetrieb

- HomeKit Bridge
- Manon behält Apple Home als vertraute Oberfläche
- schrittweise Migration der Geräte nach Home Assistant

### Phase 3 – Komfortszenen

- Morgenroutine
- Nachtmodus
- Abwesenheit
- Manon-Komfort
- Heiz- und Warmwasserlogik

### Phase 4 – Vorräte

- Grocy
- altes Android-Handy als Vorrats-Terminal
- Barcode-Scanning
- Mindestbestände
- Einkaufsliste

### Phase 5 – Garten und Wetterlogik

- Gartenkalender
- wetterabhängige Terminfindung
- Erinnerungen für Pflege, Aussaat, Rückschnitt

### Phase 6 – Kameras und Sicherheit

- UniFi Protect
- Kamera-Feeds im Dashboard
- Benachrichtigungen
- Anwesenheitslogik

### Phase 7 – Drohnen

- Indoor-Drohne
- Outdoor-Drohne
- stark eingeschränkter Sicherheitsmodus
- kein autonomes Risiko-Verhalten
