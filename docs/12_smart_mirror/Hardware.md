# Smart Mirror Badezimmer – Hardware-Setup

## Komponenten

| Komponente | Modell | Hinweise |
|-----------|--------|---------|
| **Rasberry Pi** | Pi 5 (8GB oder 16GB) | 2,4 GHz Quad-Core, 8–16GB RAM für MagicMirror² |
| **Case** | Offizielles Raspberry Pi Case für Pi 5 | Alu-Rahmen mit integriertem Lüfter, GPIO-Zugang |
| **Monitor** | Raspberry Pi Monitor 15.6" (1920×1080) | HDMI + USB-C Power, Kickstand für Wandmontage, VESA 100mm |
| **Stromversorgung** | Offizielle RP5 Power Supply (27W) | 5V 5A, USB-C, separates Netzteil empfohlen |
| **Spionspiegel** | 2-Wege-Spiegel, min. 600×400mm | Silber beschichtet, möglichst 50/50 Reflexion/Durchlässigkeit |
| **Rahmen & Montage** | Aluminium-Profil oder Holzrahmen | VESA-Montage des Monitors hinter den Spiegel |
| **Netzwerk** | WiFi 6 integriert | Badezimmer-WiFi-Abdeckung prüfen |
| **Kabel** | HDMI 2.1 (2m), USB-C Power (2m) | Für flexible Montage hinter Spiegel |

## Aufbau (Badezimmer-spezifisch)

```
┌──────────────────┐
│  Spionspiegel    │  ← Vorderseite (sieht aus wie normaler Spiegel)
├──────────────────┤
│  Monitor         │  ← Dahinter (HDMI-Anschluss nach links/rechts)
├──────────────────┤
│  Pi 5 Case       │  ← Darunter oder seitlich, nach unten orientiert
│  (mit Lüfter)    │     für optimale Luftzirkulation
└──────────────────┘
```

## Montage-Details

### Spionspiegel-Spezifikationen

- **Größe:** Mindestens 60×50cm empfohlen (Badezimmer-Standard)
- **Beschichtung:** 50% Reflexion / 50% Transmission (ideales Gleichgewicht)
- **Befestigung:** 4 Punkte mit Gummidichtungen (verhindert Feuchtigkeitsschäden)
- **Rand-Versiegelung:** Silikonversiegelung an allen Kanten (Badezimmer-Feuchte)

### Monitor-Positionierung

- Monitor exakt hinter Spiegel platzieren, max. 5cm Abstand
- **VESA 100mm** zur Wandmontage nutzen (Standard bei Pi Monitor)
- Rahmen mit ca. 2cm Rand rundherum, damit Spiegel leicht nach vorne geneigt ist (optimale Sichtbarkeit)
- Lüftungsöffnungen oben und unten des Monitors freihalten (Dampf)

### Raspberry Pi Positionierung

- **Unterhalb oder seitlich** des Monitors
- Case mit integriertem Lüfter nach unten orientiert (Kühlluft aus)
- Mindestens 10cm Abstand zu Feuchtigkeitsquellen (Dusche, Badewanne)
- Kabelführung in Kunststoff-Leitkanälen (Spritzwasser-Schutz)

## Feuchtigkeitsschutz (Critical für Badezimmer!)

- **Silikonversiegelung** an allen Spiegel-Rändern
- **IP54-Gehäuse oder Trockenbox** um den Pi (optional, empfohlen)
- **Belüftung:** Lüfter des Pi läuft **kontinuierlich**, auch im Standby
- **Lüftungsgitter** oben und unten freihalten (mindestens 5cm)
- **Feuchtemessung:** Optionaler Feuchte-Sensor an HA angebunden (Warnung bei >75% RH)

## Stromversorgung

**Szenario 1: Neues Netzteil (empfohlen)**
- Offizielle 27W Power Supply separat für den Pi
- USB-C vom Netzteil → Pi
- Monitor-USB-C → zweites kleines 5V-Netzteil (1,5A)
- Beide Netzteile **hinter dem Spiegel** in Kunststoff-Verteilungskasten

**Szenario 2: Pi-USB Power den Monitor**
- Monitor-USB-C → Pi USB
- Limitiert aber: Monitor läuft dann mit 60% Helligkeit (Pi kann max. 1,5A liefern)
- Vorteil: Nur ein Netzteil nötig

Empfehlung: **Szenario 1** für volle Helligkeit und separate Last-Verteilung.

## Kabelrouting

```
┌─ Monitor ──────┐
│   HDMI-Port    │──→ HDMI 2.1 Kabel (2m) →──┐
│   USB-C Power  │──→ zu eigenes PSU            │
└────────────────┘                            │
                                              ↓
                                    ┌─ Pi 5 ──────┐
                                    │  HDMI-Port  │
                                    │  USB-C      │
                                    │  Power      │
                                    └─────────────┘
```

Alle Kabel **hinter dem Spiegel** – nur Power-Kabel nach außen zu Steckdose.

## WiFi-Empfang

- Badezimmer-Wände often Kacheln/Beton → WiFi-Signal prüfen vor Montage
- Falls schwach: **WiFi-Repeater** oder **Mesh-Netzwerk** erweitern
- Alternative: **Ethernet über Stromleitungen (PowerLAN)** wenn verfügbar

## Trocknungstest vor Installation

1. Monitor 48 Stunden im feuchten Badezimmer laufen lassen
2. Nach jeder Dusche/Bad auf Kondensation prüfen
3. Spiegel-Ränder auf Feuchtigkeitseinzug kontrollieren
4. Nur wenn alles trocken bleibt: finale Montage

## BOM (Bill of Materials)

| Artikel | Preis (ca.) | Quelle |
|---------|------------|--------|
| Raspberry Pi 5 (8GB) | 60 € | raspberrypi.com |
| Offizielles Case Pi 5 | 15 € | raspberrypi.com |
| Raspberry Pi Monitor | 100 € | raspberrypi.com / local |
| Power Supply 27W | 15 € | raspberrypi.com |
| Monitor Power Supply | 10 € | Amazon / local |
| Spionspiegel (60×50cm) | 50–100 € | Glas-Spezialist lokal |
| Alu-Rahmen + Montage | 30–50 € | Baumarkt |
| HDMI/USB-C Kabel | 15 € | Amazon |
| Silikon + Dichtungen | 10 € | Baumarkt |
| **Summe** | **305–400 €** | |

## Nächste Schritte

1. Spionspiegel-Größe und -Bezug klären (lokal oder online)
2. Rahmen-Design skizzieren (Fensterrahmen-Stil, modern, etc.)
3. Feuchtigkeits-Messpunkte im Badezimmer testen (vor Montage)
4. MagicMirror²-Installation (siehe `12_smart_mirror/installation.md`)
