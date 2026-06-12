# 06 – Hermes mit Home Assistant verbinden

> Ziel: Die auf **JarvisCore** installierte Hermes-CLI-Instanz als User `jarvis` kontrolliert mit **Home Assistant / JarvisMansion** verbinden.
>
> Stand: 2026-06-12

---

## 0. Ausgangspunkt

Diese Anleitung setzt voraus, dass folgende Schritte abgeschlossen sind:

```text
04 – JarvisCore-VM aus Template erstellen
05 – Hermes CLI auf JarvisCore installieren
```

Erwarteter Zustand:

```text
VM-Name:       JarvisCore
Hostname:      jarvis-core
FQDN:          jarvis-core.multiversum.network
Admin-User:    marc
Runtime-User:  jarvis
Hermes:        CLI-only als User jarvis installiert
Hermes Home:   /home/jarvis/.hermes
Docker:        aus Template vorhanden, aber für diese HA-Anbindung nicht erforderlich
HA-VM:         JarvisMansion / Home Assistant OS
```

Wichtig:

```text
Hermes läuft nicht innerhalb von Home Assistant.
Hermes steuert Home Assistant nicht über Docker.
Die Verbindung erfolgt über die offizielle Hermes-Home-Assistant-Integration.
Die technische Basis sind HASS_TOKEN, HASS_URL, REST API und optional WebSocket-Gateway.
```

Nicht Bestandteil dieser Anleitung:

```text
JARVIS Persona finalisieren
Skills installieren
Produktive HA-Aktionen ausführen
Tür-/Garage-/Alarm-/Sicherheitsaktionen
Hermes Desktop Remote-Verbindung
```

---

## 1. Architekturentscheidung

Zielbild:

```text
Proxmox
├── VM: JarvisMansion
│   └── Home Assistant OS
│
└── VM: JarvisCore
    └── Hermes CLI als User jarvis
        └── Home-Assistant-Integration über API
```

Sicherheitslinie:

```text
JarvisCore ist die VM-Grenze.
Hermes darf HA analysieren.
Hermes darf HA später kontrolliert steuern.
Home Assistant bleibt das ausführende System.
```

Für diese Phase gilt ausdrücklich:

```text
Hermes darf Home Assistant sehen.
Hermes soll noch keine produktiven Aktionen frei ausführen.
```

---

## 2. Home Assistant URL festlegen

Die Home-Assistant-URL muss von JarvisCore erreichbar sein.

Beispiele:

```text
http://homeassistant.local:8123
http://jarvismansion.local:8123
http://192.168.x.x:8123
```

Für die Dokumentation verwenden wir als Platzhalter:

```text
HASS_URL=http://192.168.x.x:8123
```

Auf JarvisCore als `marc` testen:

```bash
curl -I http://192.168.x.x:8123
```

Eine Antwort von Home Assistant reicht. Ein Redirect oder eine Login-Seite ist okay.

---

## 3. Long-Lived Access Token in Home Assistant erstellen

In Home Assistant:

```text
Profil öffnen
→ Sicherheit / Long-lived access tokens
→ Token erstellen
→ Name: Hermes Agent JarvisCore
→ Token kopieren
```

Hinweise:

```text
Den Token nur einmal kopieren.
Nicht in normale Markdown-Dokumentation eintragen.
Nicht in Git committen.
Nicht in Screenshots zeigen.
```

Empfohlener Token-Name:

```text
Hermes Agent JarvisCore
```

Optional später sauberer:

```text
Eigenen HA-Benutzer für Hermes/JARVIS anlegen.
Token aus diesem Benutzerprofil erzeugen.
```

Für den ersten Test reicht ein Long-Lived Access Token aus einem bestehenden administrativen HA-Profil, solange wir noch keine produktiven Aktionen ausführen.

---

## 4. API-Zugriff von JarvisCore testen

Auf JarvisCore als `marc`:

```bash
export HASS_URL="http://192.168.x.x:8123"
export HASS_TOKEN="PASTE_TOKEN_HERE"
```

Dann testen:

```bash
curl -s \
  -H "Authorization: Bearer $HASS_TOKEN" \
  -H "Content-Type: application/json" \
  "$HASS_URL/api/"
```

Erwartung:

```json
{"message":"API running."}
```

Falls `401 Unauthorized`:

```text
Token falsch, gekürzt oder nicht als Long-Lived Access Token erstellt.
```

Falls keine Verbindung:

```text
HASS_URL, DNS, IP, Port 8123 oder Firewall prüfen.
```

Danach die Shell-Variablen wieder verwerfen:

```bash
unset HASS_TOKEN
unset HASS_URL
```

---

## 5. Zur Hermes-Runtime wechseln

```bash
sudo -iu jarvis
```

Prüfen:

```bash
whoami
echo "$HOME"
ls -la ~/.hermes
```

Erwartung:

```text
whoami = jarvis
HOME   = /home/jarvis
Hermes = /home/jarvis/.hermes
```

---

## 6. `.env` für Hermes anlegen

Die offizielle Hermes-Home-Assistant-Integration liest Zugangsdaten aus:

```text
/home/jarvis/.hermes/.env
```

Datei erstellen:

```bash
nano ~/.hermes/.env
```

Inhalt:

```env
# Home Assistant / JarvisMansion
HASS_TOKEN=PASTE_LONG_LIVED_ACCESS_TOKEN_HERE
HASS_URL=http://192.168.x.x:8123
```

Wichtig:

```text
Keine Anführungszeichen um HASS_URL.
Keine Leerzeichen vor oder nach dem Gleichheitszeichen.
Token nicht umbrechen.
```

Rechte setzen:

```bash
chmod 600 ~/.hermes/.env
ls -la ~/.hermes/.env
```

Erwartung:

```text
-rw------- 1 jarvis jarvis ... /home/jarvis/.hermes/.env
```

---

## 7. Hermes-Konfiguration prüfen

```bash
hermes config
```

Optional:

```bash
hermes config check
```

Wichtig:

```text
HASS_TOKEN ist ein Secret und gehört in .env.
HASS_URL kann ebenfalls in .env bleiben.
Nicht in normale Dokumentation kopieren.
```

---

## 8. Home-Assistant-Tools grundsätzlich aktivieren

Hermes aktiviert das Home-Assistant-Toolset automatisch, sobald `HASS_TOKEN` gesetzt ist.

Die relevanten Tools sind:

```text
ha_list_entities
ha_get_state
ha_list_services
ha_call_service
```

Für diese Anleitung nutzen wir nur lesende beziehungsweise ungefährliche Tests:

```text
ha_list_entities
ha_get_state
ha_list_services
```

`ha_call_service` wird in dieser Anleitung nur erwähnt, aber nicht produktiv verwendet.

---

## 9. Hermes interaktiv starten und HA abfragen

Als User `jarvis`:

```bash
hermes
```

Dann innerhalb der Hermes-Session sinngemäß testen:

```text
List Home Assistant entities in the sensor domain.
```

Danach:

```text
What services are available for the light domain in Home Assistant?
```

Optional mit einer realen Entity-ID:

```text
Get the current state of sensor.DEINE_ENTITY_ID.
```

Erwartung:

```text
Hermes erkennt Home Assistant.
Hermes kann Entitäten und Services lesen.
Keine produktive Aktion wurde ausgeführt.
```

Wenn Hermes die HA-Tools nicht sieht:

```bash
cat ~/.hermes/.env
hermes config check
```

Dann Hermes komplett neu starten. Änderungen an `.env` werden erst beim Neustart der Hermes-Session beziehungsweise des Gateways übernommen.

---

## 10. Gateway-Konfiguration vorbereiten

Hermes kann Home-Assistant-Events per WebSocket empfangen. Standardmäßig werden aber keine Events weitergereicht, solange keine Filter gesetzt sind.

Für die erste Verbindung konfigurieren wir bewusst eine enge Watchlist.

Datei öffnen:

```bash
nano ~/.hermes/config.yaml
```

Folgenden Abschnitt ergänzen oder anpassen:

```yaml
platforms:
  homeassistant:
    enabled: true
    extra:
      watch_domains:
        - binary_sensor
        - climate
        - alarm_control_panel
      watch_entities: []
      ignore_entities:
        - sensor.uptime
        - sensor.cpu_usage
        - sensor.memory_usage
      cooldown_seconds: 30
```

Wichtig:

```text
watch_all nicht aktivieren.
Keine breite sensor-Watchlist aktivieren.
Keine light-/switch-Watchlist in Phase 1.
```

Falls `config.yaml` bereits andere Inhalte enthält, den bestehenden Inhalt nicht blind überschreiben. Den Abschnitt sauber integrieren.

---

## 11. Gateway manuell testen

Als User `jarvis`:

```bash
hermes gateway
```

Erwartung:

```text
Hermes Gateway startet.
Home Assistant wird als Plattform verbunden.
Keine Event-Flut erscheint.
```

Dann in Home Assistant testweise einen ungefährlichen Zustand ändern, zum Beispiel ein Fenster-/Türkontakt oder einen Test-Binary-Sensor, falls vorhanden.

Beenden mit:

```text
CTRL+C
```

Noch kein Autostart einrichten.

---

## 12. Schreibende Aktionen bleiben gesperrt durch Betriebsregel

Technisch kann Hermes über `ha_call_service` Home-Assistant-Services auslösen.

Für JARVIS gilt aber ab sofort:

```text
Keine freien direkten Aktionen in Phase 1.
Keine Tür-/Garage-/Alarm-/Lock-Aktionen.
Keine Heizung-/Klima-Daueränderungen ohne Bestätigung.
Keine beliebigen switch.*-Aktionen.
Produktive Aktionen später bevorzugt über script.jarvis_*.
```

Erlaubt für spätere Tests:

```text
script.jarvis_test_notification
script.jarvis_statusbericht
script.jarvis_light_test
```

Diese Skripte werden erst in einer späteren Anleitung in Home Assistant angelegt.

---

## 13. Optionaler Minimaltest mit persistent notification

Nur wenn bewusst eine ungefährliche Schreibaktion getestet werden soll:

```text
Bitte erst in Home Assistant ein harmloses script.jarvis_test_notification anlegen.
Dann über Hermes ausschließlich dieses Script auslösen.
```

Beispiel-Ziel für später:

```text
ha_call_service(domain="script", service="turn_on", entity_id="script.jarvis_test_notification")
```

In dieser Anleitung wird dieser Test noch nicht vorausgesetzt.

---

## 14. Abschlussprüfung

Als User `jarvis`:

```bash
ls -la ~/.hermes/.env
hermes config check
```

Optional:

```bash
hermes doctor
```

Erwarteter Zustand:

```text
HASS_TOKEN und HASS_URL liegen in /home/jarvis/.hermes/.env.
Hermes kann Home Assistant erreichen.
Hermes kann Entitäten/Services lesen.
Gateway wurde manuell getestet oder bewusst noch nicht gestartet.
Keine produktiven HA-Aktionen wurden ausgeführt.
```

---

## 15. Zurück zu marc

```bash
exit
```

Prüfen:

```bash
whoami
```

Erwartung:

```text
marc
```

---

## 16. Proxmox-Snapshot erstellen

Auf dem Proxmox-Host:

```bash
qm snapshot 101 after-hermes-ha-connection --description "Hermes CLI connected to Home Assistant via HASS_TOKEN/HASS_URL. Read tests completed. No JARVIS persona or productive actions configured yet."
```

Alternativ über Proxmox UI:

```text
VM 101 JarvisCore
→ Snapshots
→ Take Snapshot
```

Snapshot-Name:

```text
after-hermes-ha-connection
```

Beschreibung:

```text
Hermes mit Home Assistant verbunden. HASS_TOKEN/HASS_URL gesetzt. Lesetests durchgeführt. Gateway optional manuell getestet. Noch keine JARVIS Persona, keine Skills und keine produktiven Aktionen.
```

---

## 17. Stopppunkt

Erledigt:

```text
Home Assistant URL festgelegt
Long-Lived Access Token erstellt
HA API per curl getestet
/home/jarvis/.hermes/.env angelegt
Hermes HA-Tools geprüft
Gateway-Filter vorbereitet
Gateway optional manuell getestet
Snapshot after-hermes-ha-connection erstellt
```

Nicht erledigt:

```text
JARVIS Persona noch nicht übernommen
SOUL.md noch nicht final gesetzt
Skills noch nicht installiert
script.jarvis_* noch nicht produktiv genutzt
Hermes Gateway noch nicht als Service eingerichtet
Hermes Desktop Remote-Verbindung noch nicht getestet
```

Nächster Schritt:

```text
07 – Hermes zu JARVIS machen: Persona, SOUL und Betriebsregeln
```
