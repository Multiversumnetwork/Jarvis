# 07 – Hermes zu JARVIS machen: Persona, SOUL und Betriebsregeln

> Ziel: Die bestehende **JARVIS Persona** sauber in Hermes übernehmen und erste verbindliche Betriebsregeln für Home Assistant / JarvisMansion festlegen.
>
> Stand: 2026-06-12

---

## 0. Ausgangspunkt

Diese Anleitung setzt voraus, dass folgende Schritte abgeschlossen sind:

```text
04 – JarvisCore-VM aus Template erstellen
05 – Hermes CLI auf JarvisCore installieren
06 – Hermes mit Home Assistant verbinden
```

Erwarteter Zustand:

```text
VM-Name:       JarvisCore
Hostname:      jarvis-core
Admin-User:    marc
Runtime-User:  jarvis
Hermes Home:   /home/jarvis/.hermes
HA-Verbindung: HASS_TOKEN/HASS_URL in /home/jarvis/.hermes/.env
JARVIS Persona: existiert bereits als separate Quelle
```

Wichtig:

```text
Diese Anleitung erfindet keine neue JARVIS Persona.
Die bestehende Persona wird übernommen, geprüft und mit Betriebsregeln ergänzt.
```

Nicht Bestandteil dieser Anleitung:

```text
Neue Skills produktiv installieren
Home-Assistant-Skripte anlegen
Autostart für Hermes Gateway einrichten
Hermes Desktop Remote-Verbindung testen
Produktive Hausaktionen freigeben
```

---

## 1. Rolle von SOUL.md verstehen

Hermes nutzt:

```text
/home/jarvis/.hermes/SOUL.md
```

als primäre Agentenidentität.

Für JARVIS bedeutet das:

```text
SOUL.md = Wer JARVIS ist
config.yaml = Wie Hermes technisch arbeitet
.env = Secrets und Tokens
skills/ = Fachlogik und wiederverwendbare Fähigkeiten
```

Die bestehende JARVIS Persona gehört also nach:

```text
/home/jarvis/.hermes/SOUL.md
```

Nicht nach:

```text
/opt/jarvis-core/data
/opt/data
/root/.hermes
/home/marc/.hermes
```

---

## 2. Zur Hermes-Runtime wechseln

Als Admin `marc`:

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

## 3. Bestehende SOUL.md sichern

Falls Hermes bereits eine SOUL.md erzeugt hat:

```bash
if [ -f ~/.hermes/SOUL.md ]; then
  cp ~/.hermes/SOUL.md ~/.hermes/SOUL.md.bak-$(date +%Y%m%d-%H%M%S)
fi
```

Prüfen:

```bash
ls -la ~/.hermes/SOUL.md*
```

---

## 4. Bestehende JARVIS Persona übernehmen

Die JARVIS Persona existiert bereits in der Projektdokumentation.

Es gibt zwei saubere Varianten:

```text
Variante A: Persona-Inhalt direkt nach ~/.hermes/SOUL.md kopieren
Variante B: Persona-Datei nach ~/.hermes/persona/JARVIS-PERSONA.md kopieren und SOUL.md daraus bewusst zusammensetzen
```

Empfohlen für Phase 1:

```text
Variante A – direkte Übernahme nach SOUL.md
```

Datei öffnen:

```bash
nano ~/.hermes/SOUL.md
```

Dann die bestehende JARVIS Persona vollständig einfügen.

Wichtig:

```text
Keine neue Persona formulieren.
Keine Pepper-Persona übernehmen.
Keine Raymond-Persona übernehmen.
Keine privaten Token oder URLs in SOUL.md eintragen.
```

---

## 5. Betriebsregeln ergänzen

Unterhalb der bestehenden JARVIS Persona einen Abschnitt ergänzen:

```markdown
## Betriebsregeln für JarvisCore und Home Assistant

- Du bist JARVIS, der analytische Home-Assistant- und Haussteuerungs-Agent für Marc.
- Du läufst auf JarvisCore und kommunizierst mit Home Assistant / JarvisMansion ausschließlich über die offizielle Home-Assistant-Integration von Hermes.
- Home Assistant bleibt das ausführende System. Du bist Analyse-, Entscheidungs- und Assistenzschicht.
- Lies Zustände zuerst, bevor du Aktionen vorschlägst oder ausführst.
- Erkläre kritische Empfehlungen kurz und sachlich.
- Führe keine sicherheitsrelevanten Aktionen ohne explizite Bestätigung aus.
- Öffne keine Türen, Schlösser, Tore oder Garagen ohne explizite Bestätigung.
- Deaktiviere keine Alarmanlage ohne explizite Bestätigung.
- Ändere keine langfristigen Heizungs-, Klima- oder Energieeinstellungen ohne explizite Bestätigung.
- Nutze für produktive Aktionen bevorzugt Home-Assistant-Skripte mit dem Namensschema `script.jarvis_*`.
- Nutze direkte `ha_call_service`-Aufrufe nur für ungefährliche Tests oder wenn Marc es ausdrücklich erlaubt.
- Wenn eine Aktion unklar, riskant oder mehrdeutig ist, analysiere und frage nach, statt zu handeln.
- Bei Fehlern in Home Assistant gib eine klare Diagnose und keine spekulative Reparaturaktion.
```

Hinweis:

```text
Dieser Abschnitt ersetzt keine spätere Skill-Policy.
Er ist die erste globale Sicherheits- und Verhaltensschicht.
```

---

## 6. Dateirechte prüfen

```bash
chmod 600 ~/.hermes/SOUL.md
ls -la ~/.hermes/SOUL.md
```

Erwartung:

```text
-rw------- 1 jarvis jarvis ... /home/jarvis/.hermes/SOUL.md
```

---

## 7. Hermes-Konfiguration prüfen

```bash
hermes config check
```

Optional:

```bash
hermes doctor
```

Falls Hermes eine Migration anbietet:

```bash
hermes config migrate
```

Nur durchführen, wenn die Ausgabe plausibel ist. Keine Secrets in Klartext bestätigen, wenn unklar ist, wohin sie geschrieben werden.

---

## 8. Erste JARVIS-Identitätsprüfung

Hermes interaktiv starten:

```bash
hermes
```

Dann fragen:

```text
Wer bist du und welche Rolle hast du in dieser Umgebung?
```

Erwartung:

```text
Hermes antwortet als JARVIS.
JARVIS beschreibt JarvisCore als Brain-/Agenten-VM.
JARVIS beschreibt Home Assistant / JarvisMansion als ausführende Haussteuerung.
JARVIS erwähnt keine Pepper- oder Raymond-Rolle.
```

Danach testen:

```text
Darfst du die Garage öffnen, wenn ein Sensor komisch aussieht?
```

Erwartung:

```text
JARVIS verweigert direkte Aktion ohne explizite Bestätigung.
JARVIS schlägt Diagnose/Analyse vor.
```

---

## 9. Erste HA-Analyse mit Persona testen

Innerhalb der Hermes-Session:

```text
Analysiere den aktuellen Home-Assistant-Zustand grob, aber führe keine Aktionen aus.
```

Erwartung:

```text
JARVIS liest Zustände.
JARVIS führt keine ha_call_service-Aktion aus.
JARVIS gibt eine strukturierte Analyse zurück.
```

Optional enger:

```text
Liste die binary_sensor-Entitäten aus Home Assistant und fasse sicherheitsrelevante Zustände zusammen. Keine Aktionen ausführen.
```

---

## 10. Keine produktiven Aktionen in dieser Anleitung

Nicht testen:

```text
Licht wirklich schalten
Garage öffnen
Tür öffnen
Alarm ändern
Heizung ändern
Schalter toggeln
Automationen aktivieren/deaktivieren
```

Erlaubt:

```text
Entitäten lesen
Services auflisten
Einzelne Zustände lesen
Risikoanalyse formulieren
Empfehlungen ausgeben
```

---

## 11. Skills-Verzeichnis vorbereiten

Noch keine Skills installieren, aber Verzeichnis prüfen:

```bash
mkdir -p ~/.hermes/skills
ls -la ~/.hermes/skills
```

Vorgesehene spätere Skills:

```text
jarvis-home-assistant-policy
jarvis-security-check
jarvis-energy-manager
jarvis-stromee
jarvis-maintenance-diagnostics
```

Hinweis:

```text
Die globale SOUL.md ist nicht der richtige Ort für komplexe Fachlogik.
Dafür nutzen wir später Skills.
```

---

## 12. Memory-Verzeichnis prüfen

```bash
mkdir -p ~/.hermes/memories
ls -la ~/.hermes/memories
```

Noch nicht automatisch befüllen.

Spätere Memory-Themen:

```text
Hausstruktur Udange
Geräte und Räume
Stromee / BMW i3
OpenWB / Energie
Sicherheitslogik
Wartungsnotizen
```

---

## 13. Stopptest

Hermes erneut starten:

```bash
hermes
```

Kurzer Testdialog:

```text
Du bist JARVIS. Was darfst du aktuell mit Home Assistant tun?
```

Erwartete Kernaussage:

```text
Ich darf Home Assistant analysieren und Zustände lesen.
Produktive Aktionen führe ich nur kontrolliert und mit Freigabe aus.
Kritische Aktionen benötigen explizite Bestätigung.
```

---

## 14. Zurück zu marc

```bash
exit
```

Falls noch eine Hermes-Session offen ist, diese vorher sauber beenden.

Dann:

```bash
whoami
```

Erwartung:

```text
marc
```

---

## 15. Proxmox-Snapshot erstellen

Auf dem Proxmox-Host:

```bash
qm snapshot 101 after-jarvis-soul-policy --description "Existing JARVIS persona added to Hermes SOUL.md. Initial Home Assistant safety policy documented. No productive HA actions or skills configured yet."
```

Alternativ über Proxmox UI:

```text
VM 101 JarvisCore
→ Snapshots
→ Take Snapshot
```

Snapshot-Name:

```text
after-jarvis-soul-policy
```

Beschreibung:

```text
Bestehende JARVIS Persona in /home/jarvis/.hermes/SOUL.md übernommen. Erste HA-Betriebsregeln ergänzt. Keine produktiven Aktionen, keine Skills und kein Gateway-Autostart eingerichtet.
```

---

## 16. Stopppunkt

Erledigt:

```text
Bestehende SOUL.md gesichert
Bestehende JARVIS Persona nach /home/jarvis/.hermes/SOUL.md übernommen
Betriebsregeln für Home Assistant ergänzt
JARVIS-Identität getestet
HA-Analyse ohne Aktionen getestet
skills/ und memories/ vorbereitet
Snapshot after-jarvis-soul-policy erstellt
```

Nicht erledigt:

```text
Skills noch nicht installiert
Memory noch nicht befüllt
script.jarvis_* noch nicht angelegt
Produktive HA-Aktionen noch nicht freigegeben
Hermes Gateway noch nicht als Dienst eingerichtet
Hermes Desktop Remote-Verbindung noch nicht getestet
```

Nächster Schritt:

```text
08 – JARVIS Skills und kontrollierte Home-Assistant-Tests
```
