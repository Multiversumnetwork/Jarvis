# JARVIS

## Digitales Betriebssystem für Haus und Alltag

---

# Mission

JARVIS ist das zentrale digitale Betriebs- und Assistenzsystem des Hauses.

Der Nutzer interagiert ausschließlich mit JARVIS.

Alle technischen Komponenten arbeiten unsichtbar im Hintergrund und werden über eine einheitliche Oberfläche sowie über natürliche Sprachsteuerung bereitgestellt.

---

# Leitprinzip

> Der Nutzer sieht JARVIS. Nicht die Infrastruktur.

JARVIS ist das Produkt.

Alle anderen Systeme sind Komponenten.

Der Nutzer interagiert ausschließlich mit JARVIS.

Home Assistant, Mealie, Grocy, Frigate, OpenWB und weitere Dienste werden als interne Komponenten betrachtet und treten gegenüber dem Nutzer nicht eigenständig in Erscheinung.

---

# Persönlichkeit

## Charakter

JARVIS ist:

- kompetent
- ruhig
- präzise
- zuverlässig
- sachlich
- professionell

Persona als einzelnes Dokument: PERSONA.md
---

## Humor

JARVIS darf gelegentlich trockenen, intelligenten Humor einsetzen.

Der Humor soll:

- selten verwendet werden
- situationsbezogen sein
- niemals die eigentliche Information verdrängen
- subtil und hochwertig wirken

Inspiration:

- J.A.R.V.I.S. aus dem Marvel-Universum
- britischer Understatement-Humor
- höfliche Ironie

Beispiel:

> „Der Kaffeevorrat liegt bei zwei Packungen. Die Lage ist unter Kontrolle.“

---

# Künstliche Intelligenz

## Large Language Model

Aktueller Favorit:

- Ministral 3 14B Instruct (Infomaniak)

Aufgaben:

- Interpretation von Sprachbefehlen
- Kontextverständnis
- Zusammenführung von Informationen
- Empfehlungen
- Dialogführung
- Steuerung von JARVIS-Funktionen

---

# Hardware-Plattform

## JARVIS Core

### Primärsystem

Intel NUC 10

Hostname: JARVIS 

Ausstattung:

- Intel Core i7-10710U
- 16 GB DDR4 RAM
- 512 GB M.2 NVMe SSD
- Dauerbetrieb 24/7

Rolle:

Zentrale Hardware-Plattform für JARVIS.

Funktion:

- Proxmox
- Home Assistant OS (JARVIS Core)
- Mealie/Tandoor (Rezepte & Kochplanung)
- Grocy (Lagerverwaltung)
- Frigate (optional)
- Sprachsystem
- weitere JARVIS-Komponenten

---

# Sprachhardware

## Nabu Casa Voice Preview Edition

Primäre Sprachschnittstelle von JARVIS.

### Sprachsystem

Wake Word:

- OpenWakeWord

Speech-to-Text:

- Whisper

Text-to-Speech:

- Deutsche J.A.R.V.I.S.-Stimme

Large Language Model:

- Ministral 3 14B Instruct

Aufgaben:

- Wake Word Erkennung
- Spracheingabe
- Sprachausgabe
- Dialogführung
- Interaktion mit JARVIS

Geplante Positionen:

- Küche
- später Satelliten in weiteren Räumen

---

# Dashboard-Hardware

## Küchen-Dashboard

Großes fest montiertes Samsung Tablet.

Funktion:

- Permanentes JARVIS Dashboard
- Hausstatus
- Energieübersicht
- Küchenplanung
- Essensplan
- Einkaufslisten
- Kamerabilder
- Wetter
- Kalender

Ziel:

Der Nutzer soll möglichst alle alltäglichen Interaktionen über das Dashboard oder die Sprachsteuerung durchführen können.

---

## Dashboard-Prinzip

### Standardansicht

JARVIS Screensaver

Anzeige:

- Uhrzeit
- Wetter
- Hausstatus
- Energiefluss
- dezente Animationen im JARVIS-Stil

---

### Navigation

Per Touch oder Sprache.

Bereiche:

- Home
- Energie
- Küche
- Haushalt
- Garten
- Sicherheit

---

# Kernbereiche

## Haussteuerung

- Licht
- Steckdosen
- Rollläden
- Heizung
- Klimatisierung
- Szenen

---

## Energie

- Photovoltaik
- Batteriespeicher
- Wärmepumpe
- OpenWB
- Elektrofahrzeuge
- Verbrauchsanalysen

---

## Küche

- Rezeptverwaltung
- Essensplanung
- Einkaufslisten
- Zutatenverfügbarkeit

---

## Haushalt

- Inventarverwaltung
- Vorräte
- Batterien
- Equipment
- Verbrauchsmaterialien
- Müllkalender

---

## Garten

- Bewässerung
- Wetter
- Mähroboter
- Pflanzplanung

---

## Sicherheit

- Kameras
- Sensoren
- Benachrichtigungen
- Alarme

---

# Systemkomponenten

## Proxmox Basis

## Home Assistant OS

JARVIS Core

Verantwortlich für:

- Dashboards
- Integrationen
- Automationen
- Benutzeroberfläche
- Sprachintegration

---

## Mealie / Tandoor

JARVIS Kitchen Engine
(https://github.com/alexbelgium/hassio-addons/tree/master/mealie#installation)
Verantwortlich für:

- Rezepte
- Essensplanung
- Einkaufslisten

---

## Grocy

JARVIS Inventory Engine

Verantwortlich für:

- Vorräte
- Lagerbestände
- Batterien
- Equipment
- Verbrauchsmaterialien

Nicht verwendet:

- Rezepte
- Haushaltsaufgaben
- Kalenderfunktionen

---

## Frigate

JARVIS Vision Engine

Verantwortlich für:

- Kameras
- Objekterkennung
- Ereigniserkennung

Optional je nach verfügbarer Hardwareleistung.

---

## OpenWB

JARVIS Charging Engine

Verantwortlich für:

- Wallbox
- Ladeplanung
- Fahrzeugdaten

---

# Betriebsmodell

## Dauerbetrieb

JARVIS läuft permanent.

Ziel:

- Sofortige Verfügbarkeit
- Permanente Sprachsteuerung
- Keine Startzeiten
- Hohe Zuverlässigkeit

---

## Lokaler Betrieb

JARVIS soll möglichst unabhängig von externen Cloud-Diensten arbeiten.

Bevorzugt werden:

- lokale Sprachverarbeitung
- lokale Automationen
- lokale Datenspeicherung

Cloud-Dienste werden nur eingesetzt, wenn ein klarer Mehrwert besteht.

---

# Backup-Strategie

## Grundsatz

> Keine Konfigurationsänderung ohne Backup.
> Kein Update ohne Snapshot.

- Clonezilla???

---

## Tägliche Sicherung

- Home Assistant Backups
- Mealie Datenbank
- Grocy Datenbank
- Sprachkonfiguration
- Dashboard-Konfiguration

---

## Backup-Ziel

Primär:

- Synology NAS

Sekundär:

- Offsite-Backup

---

## Vor größeren Änderungen

- Manuelle Snapshots
- Konfigurationssicherung
- Dokumentation der Änderungen

---

# Geplante Erweiterungen

## Garten

- DUM-E (Segway Navimow i208)
- Integration in JARVIS
- Wetter- und Bewässerungslogik

---

## Inventarverwaltung

- Altes Android-Smartphone als Barcode-Scanner
- Integration mit Grocy

---

## Sicherheit

- UniFi Protect Kamerasystem
- UniFi Smart Locks
- Mobile Überwachungsdrohne ("Iron Man Drohne")

---

## Energie

- ARC-Reaktor (PV-Anlage)
- T-ARC-Reaktor (SolvisMax Heizsystem)
- OpenWB Wallbox
- BMW i3s „Stromee“

---

# Status

Projektstatus:

Planung / Aufbau

---

## Priorität 1

- Proxmox
- Home Assistant OS
- Sprachsteuerung
- Dashboard
- Apple Home Bridge (Manon)
- Migration der Komponenten aus Apple Home
- Hue Bridge Pro

---

## Priorität 2

- Mealie / Tandoor
- Grocy
- DUM-E

---

## Priorität 3

- Szenen
- Erweiterte Automationen

---

## Priorität 4

- Frigate
- UniFi Protect
- Drohne

---

# Langfristige Vision

JARVIS entwickelt sich zum zentralen digitalen Betriebssystem des Hauses.

Er kennt:

- den Zustand des Hauses
- den Energieverbrauch
- die Vorräte
- den Essensplan
- den Garten
- die Sicherheitskomponenten
- anstehende Aufgaben

und stellt diese Informationen über Sprache und eine einheitliche Benutzeroberfläche bereit.

JARVIS ist kein einzelnes Programm.
JARVIS ist das digitale Betriebssystem des Hauses.