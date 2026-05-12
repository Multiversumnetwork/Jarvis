# Gartenkalender und Wetterlogik

## Ziel

JARVIS soll Gartenaufgaben nicht nur erinnern, sondern den passenden Zeitpunkt anhand von Wetterdaten vorschlagen.

## Beispiele

- Buchs schneiden
- Garten umgraben
- Aussaat
- Pflanzen setzen
- Rückschnitt
- Bewässerung
- Frostwarnung
- Tomaten auspflanzen
- Bohnen setzen

## Grundlogik

```text
Aufgabe
+ möglicher Zeitraum
+ Wetterbedingungen
+ Kalenderverfügbarkeit
= Terminvorschlag
```

## Beispiel: Buchs schneiden

Bedingungen:

- nicht bei Frost
- nicht bei starker Hitze
- nicht bei Starkregen
- trockenes Wetterfenster
- möglichst bewölkt
- Zeitraum abhängig von Saison

JARVIS Antwort:

```text
Für den Buchs wäre Samstagvormittag geeignet. Es bleibt trocken und die Temperaturen liegen moderat.
```

## Beispiel: Garten umgraben

Bedingungen:

- Boden nicht gefroren
- Boden nicht zu nass
- kein Starkregen danach
- genug Zeitfenster im Kalender

## Datenquellen

Geplant:

- Wetterintegration in Home Assistant
- Kalender
- manuelle Gartenaufgaben
- später Bodensensoren optional

## Kalenderstruktur

Mögliche Kategorien:

```text
Garten/Pflege
Garten/Aussaat
Garten/Ernte
Garten/Rückschnitt
Garten/Bewässerung
```

## JARVIS Aufgaben

JARVIS soll:

- anstehende Aufgaben prüfen
- Wetter abgleichen
- beste Termine vorschlagen
- Erinnerungen senden
- Aufgaben verschieben, wenn Wetter ungeeignet ist
- saisonale Vorschläge machen

## Sicherheits-/Realitätsregel

JARVIS darf nur vorschlagen.

Gartenentscheidungen bleiben bei Marc/Manon.
