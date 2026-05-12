# Vorratsverwaltung mit Grocy

## Ziel

JARVIS soll Überblick über Vorräte im Haus haben.

Schwerpunkt:

- Waschküche
- Vorratsregale
- Haushaltsreserven
- Lebensmittel
- Verbrauchsmaterial

## Software

### Grocy

Grocy wird als zentrale Warenwirtschaft genutzt.

Funktionen:

- Produkte
- Bestände
- Mindestbestände
- Einkaufsliste
- Barcodes
- Haltbarkeiten
- Verbrauch
- Haushaltsmanagement

## Hardware-Entscheidung

Statt Pi-Terminal oder festem Barcodescanner wird zunächst ein altes Android-Handy genutzt.

Gründe:

- bereits vorhanden
- Kamera integriert
- Akku
- WLAN
- Touchscreen
- mobil
- flexibel am Regal nutzbar
- weniger Wartung
- kein zusätzliches Pi-Setup

## Android statt LineageOS

Start:

- normales Android verwenden
- Gerät zurücksetzen
- nur notwendige Apps installieren
- Grocy testen
- Barcode-Scanning testen

LineageOS nur falls:

- Android zu alt ist
- Gerät offiziell unterstützt wird
- Kamera/WLAN stabil laufen
- Google-Dienste bewusst vermieden werden sollen

## Physisches Setup

Geplant:

- altes Android-Handy
- Magnethalterung an Metallregal
- Ladepunkt in der Waschküche
- optional magnetischer USB-C-Ladeadapter
- optional robuste Hülle

## Workflow Einkauf

```text
Einkauf kommt heim
→ Handy vom Regal nehmen
→ Produkt scannen
→ Menge bestätigen
→ Produkt einsortieren
→ Bestand aktualisiert
```

## Workflow Verbrauch

Optionen:

### Variante A – Manuell

Produkt beim Entnehmen scannen und Verbrauch buchen.

### Variante B – Mindestbestände

Für viele Produkte reicht eine Mindestbestandslogik.

Beispiele:

- Kaffee
- Waschmittel
- Toilettenpapier
- Küchenrolle
- Nudeln
- Reis
- Konserven
- Getränke

## JARVIS Funktionen

JARVIS soll:

- niedrige Bestände melden
- Einkaufsliste erstellen
- Produkte nach Priorität gruppieren
- Vorräte erklären
- geplante Rezepte mit Vorräten abgleichen
- Haushaltsverbrauch grob analysieren

Beispiele:

```text
Der Kaffeebestand liegt unter Mindestwert. Ich habe Kaffee zur Einkaufsliste hinzugefügt.
```

```text
Für Chili fehlen Mais und Kidneybohnen.
```

```text
Waschmittel reicht voraussichtlich noch zwei Wochen.
```

## Integration in Home Assistant

Grocy wird in Home Assistant integriert.

Ziel:

- Sensoren für kritische Bestände
- Einkaufsliste im Dashboard
- Benachrichtigungen
- Sprachabfragen über JARVIS

## Fokus

Nicht jedes Produkt muss perfekt gepflegt werden.

Priorität haben:

- Reserven
- haltbare Lebensmittel
- Verbrauchsmaterial
- Haushaltsbedarf
