# Kameras und UniFi Protect

## Ziel

Kameras werden später in JARVIS Home integriert.

Geplante Plattform:

- UniFi Protect

## Gründe

- gutes Ökosystem
- lokale Kontrolle
- Home-Assistant-Integration
- Kamerafeeds im Dashboard
- Ereignisse nutzbar für Automationen

## Mögliche Geräte

- Außenkameras
- Doorbell
- Innenkamera optional
- Garage/Werkstatt optional

## Nutzung in JARVIS

JARVIS soll:

- Kameraereignisse melden
- Status erklären
- Kamerafeeds im Dashboard anzeigen
- Tür-/Eingangsereignisse einordnen
- Anwesenheit berücksichtigen

## Datenschutz

Kameras sind sensibel.

Regeln:

- keine unnötige Innenraumüberwachung
- keine automatischen Freigaben ohne klare Regeln
- keine externen Cloud-Abhängigkeiten ohne bewusste Entscheidung
- Zugriff dokumentieren
- Benachrichtigungen sinnvoll begrenzen

## Beispiele

```text
Bewegung an der Einfahrt erkannt.
```

```text
Die Haustür-Kamera hat um 18:42 eine Person erkannt.
```

## Spätere Verknüpfungen

- Nachtmodus
- Abwesenheitsmodus
- Türstatus
- Beleuchtung
- Drohnenprojekt
