# 05 – Hermes CLI auf JarvisCore installieren

> Ziel: Hermes auf der vorbereiteten **JarvisCore-VM** nach offizieller **CLI-only Installationsweise** installieren und als dedizierter Runtime-User `jarvis` initial prüfen.
>
> Stand: 2026-06-12

---

## 0. Ausgangspunkt

Diese Anleitung setzt voraus, dass **04 – JarvisCore-VM aus Template erstellen** abgeschlossen wurde.

Erwarteter Zustand:

```text
VM-Name:      JarvisCore
Hostname:    jarvis-core
FQDN:        jarvis-core.multiversum.network
Admin-User:  marc
Runtime-User: jarvis
Docker:      bereits aus Template vorhanden
Hermes:      noch nicht installiert
```

Wichtig:

```text
Hermes Desktop wird nicht auf JarvisCore installiert.
JarvisCore bekommt nur Hermes CLI / Server-Basis.
Hermes Desktop kann später auf einem Client, z. B. Mac, laufen und sich mit der Remote-Instanz verbinden.
```

---

## 1. Benutzerkonzept

Für JarvisCore gilt:

```text
marc    = Admin-User mit sudo-/root-Rechten
jarvis  = dedizierter Runtime-/Service-User für Hermes
```

Hermes wird **nicht** als `root` und **nicht** unter `marc` installiert, sondern als User `jarvis`.

Dadurch liegt die Hermes-Konfiguration später standardmäßig unter:

```text
/home/jarvis/.hermes/
```

Wichtig:

```text
HERMES_HOME wird nicht manuell gesetzt.
Wir nutzen zunächst den offiziellen Standardpfad des Users jarvis.
```

---

## 2. Als Admin verbinden

Von einem Client aus per SSH verbinden:

```bash
ssh marc@jarvis-core
```

Falls DNS noch nicht aktiv ist:

```bash
ssh marc@<IP-DER-JARVISCORE-VM>
```

Prüfen:

```bash
hostnamectl
whoami
id jarvis
```

Erwartung:

```text
Hostname ist jarvis-core
angemeldeter User ist marc
User jarvis existiert
```

---

## 3. Grundvoraussetzungen prüfen

Die offizielle Hermes-Installation benötigt auf Nicht-Windows-Systemen primär `git`. Der Installer kümmert sich anschließend selbst um weitere Bestandteile wie Python, Node.js, ripgrep und ffmpeg.

Prüfen:

```bash
git --version
curl --version
```

Falls nötig nachinstallieren:

```bash
sudo apt update
sudo apt install -y git curl
```

Optional prüfen:

```bash
sudo -l
```

`marc` sollte sudo-Rechte haben.

---

## 4. Optional: Browser-/Playwright-Systemabhängigkeiten vorbereiten

Hermes kann Browser-Automation nutzen. Für einen dedizierten Service-User ohne sudo empfiehlt die offizielle Doku, die nötigen Chromium-Systembibliotheken einmalig als Admin zu installieren.

Für unsere erste JARVIS-Phase ist Browser-Automation **nicht zwingend notwendig**.

### Variante A – Minimaler CLI-only Start ohne Browser-Automation

Diese Variante ist für Phase 1 empfohlen:

```text
Hermes CLI installieren
Browser-Automation zunächst überspringen
Home Assistant später per Hermes-Integration anbinden
```

Dann wird später beim Installer `--skip-browser` genutzt.

### Variante B – Browser-Systemabhängigkeiten vorbereiten

Nur ausführen, wenn Browser-Automation direkt genutzt werden soll:

```bash
sudo npx playwright install-deps chromium
```

Hinweis:

```text
Dieser Schritt kann Node/npx verwenden bzw. nachziehen.
Für Phase 1 ist er nicht erforderlich.
```

---

## 5. Zum Runtime-User jarvis wechseln

```bash
sudo su - jarvis
```

Prüfen:

```bash
whoami
pwd
```

Erwartung:

```text
whoami = jarvis
pwd    = /home/jarvis
```

---

## 6. PATH für lokale User-Binaries vorbereiten

Der Hermes-Installer legt den Launcher bei einer Per-User-Installation typischerweise unter `~/.local/bin/hermes` ab.

Daher PATH für den User `jarvis` vorbereiten:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Prüfen:

```bash
echo "$PATH"
```

Der Pfad sollte enthalten:

```text
/home/jarvis/.local/bin
```

---

## 7. Hermes CLI installieren

### Empfohlene Variante für Phase 1: CLI-only ohne Browser-Automation

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash -s -- --skip-browser
```

Begründung:

```text
JarvisCore braucht zunächst nur Hermes CLI / Server-Basis.
Hermes Desktop wird nicht installiert.
Browser-Automation ist für die erste Home-Assistant-Anbindung nicht erforderlich.
```

### Alternative: CLI-only mit Browser-Automation

Nur verwenden, wenn Browser-Tools direkt benötigt werden:

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

---

## 8. Shell neu laden

Nach der Installation:

```bash
source ~/.bashrc
```

Prüfen:

```bash
which hermes
hermes --help
```

Erwartung:

```text
which hermes zeigt auf einen Pfad unter /home/jarvis/
hermes --help gibt die Hermes-Hilfe aus
```

Falls `hermes` nicht gefunden wird:

```bash
echo "$PATH"
ls -la ~/.local/bin
ls -la ~/.hermes/hermes-agent/venv/bin/hermes
```

Optional als Admin später einen Symlink setzen, falls ein systemweiter Aufruf benötigt wird:

```bash
exit
sudo ln -s /home/jarvis/.hermes/hermes-agent/venv/bin/hermes /usr/local/bin/hermes
sudo su - jarvis
```

Für Phase 1 reicht normalerweise der User-PATH.

---

## 9. Hermes-Verzeichnis prüfen

Als User `jarvis`:

```bash
ls -la /home/jarvis
ls -la /home/jarvis/.hermes
```

Erwartete Grundstruktur:

```text
/home/jarvis/.hermes/
```

Je nach Installationsstand können dort bereits Repository, Konfiguration oder weitere Unterverzeichnisse liegen.

Wichtig:

```text
Keine Dateien unter /opt/data anlegen.
Keine Hermes-Dateien unter /opt/jarvis-core/data anlegen.
Kein HERMES_HOME setzen, solange der Standardpfad /home/jarvis/.hermes genutzt wird.
```

---

## 10. Hermes Doctor ausführen

```bash
hermes doctor
```

Erwartung:

```text
Hermes erkennt die Installation.
Fehlende optionale Bestandteile werden gemeldet.
Keine kritischen Python-/Launcher-Fehler.
```

Falls diese Fehlermeldung erscheint:

```text
ModuleNotFoundError: No module named 'dotenv'
```

Dann wird vermutlich die falsche Datei mit System-Python gestartet. In diesem Fall prüfen:

```bash
which hermes
ls -la ~/.local/bin/hermes
ls -la ~/.hermes/hermes-agent/venv/bin/hermes
```

Der Launcher muss auf die Hermes-venv zeigen, nicht auf eine Repo-Quelldatei.

---

## 11. Modell-/Provider-Konfiguration noch nicht finalisieren

Für diese Anleitung wird Hermes nur installiert und technisch geprüft.

Noch nicht final konfigurieren:

```text
LLM Provider
OpenRouter API Key
Home Assistant Token
JARVIS Persona
Skills
Gateway
Remote Desktop Verbindung
```

Diese Punkte kommen in den nächsten Anleitungen.

Falls der Installer interaktiv nach Provider/API-Key fragt, kann die Konfiguration entweder minimal abgeschlossen oder später erneut aufgerufen werden:

```bash
hermes model
```

oder:

```bash
hermes setup
```

Für JARVIS ist später wahrscheinlich OpenRouter naheliegend.

---

## 12. Keine Home-Assistant-Anbindung in dieser Anleitung

Noch nicht anlegen:

```text
/home/jarvis/.hermes/.env mit HASS_TOKEN
HASS_URL
Gateway-Konfiguration
Home-Assistant-Watchlist
script.jarvis_*-Policy
```

Grund:

```text
Erst muss Hermes sauber laufen.
Danach verbinden wir Hermes mit Home Assistant kontrolliert per offizieller Hermes-Home-Assistant-Integration.
```

---

## 13. Bestehende JARVIS Persona noch nicht übernehmen

Die JARVIS Persona existiert bereits.

Sie wird in dieser Anleitung noch nicht nach:

```text
/home/jarvis/.hermes/SOUL.md
```

kopiert oder verlinkt.

Grund:

```text
Erst Hermes installieren und prüfen.
Danach JARVIS-Persona sauber übernehmen und testen.
```

---

## 14. Installation verlassen

Vom User `jarvis` zurück zu `marc`:

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

## 15. Abschlussprüfung

Als `marc` prüfen:

```bash
sudo -u jarvis bash -lc 'whoami && echo $HOME && which hermes && hermes --help >/dev/null && echo "Hermes CLI OK"'
```

Optional:

```bash
sudo -u jarvis bash -lc 'hermes doctor'
```

Erwartung:

```text
Hermes CLI läuft als User jarvis.
Hermes liegt unter /home/jarvis/.hermes.
Hermes Desktop ist nicht installiert.
Home Assistant ist noch nicht angebunden.
```

---

## 16. Proxmox-Snapshot erstellen

Auf dem Proxmox-Host:

```bash
qm snapshot 101 after-hermes-cli-install --description "Hermes CLI installed as user jarvis on JarvisCore. CLI verified. No Home Assistant connection, no JARVIS persona, no skills configured yet."
```

Alternativ über Proxmox UI:

```text
VM 101 JarvisCore
→ Snapshots
→ Take Snapshot
```

Snapshot-Name:

```text
after-hermes-cli-install
```

Beschreibung:

```text
Hermes CLI als User jarvis installiert und geprüft. Keine Home-Assistant-Anbindung, keine JARVIS Persona und keine Skills eingerichtet.
```

---

## 17. Stopppunkt

Erledigt:

```text
Hermes CLI-only Installer als User jarvis ausgeführt
Hermes Desktop nicht installiert
Hermes unter /home/jarvis/.hermes vorbereitet
PATH für jarvis geprüft
hermes --help geprüft
hermes doctor ausgeführt
Snapshot after-hermes-cli-install erstellt
```

Nicht erledigt:

```text
OpenRouter / LLM Provider noch nicht final konfiguriert
Home Assistant noch nicht angebunden
HASS_TOKEN noch nicht gesetzt
HASS_URL noch nicht gesetzt
Gateway noch nicht eingerichtet
JARVIS Persona noch nicht übernommen
Skills noch nicht eingerichtet
Hermes Remote-/Desktop-Verbindung noch nicht getestet
```

Nächster Schritt:

```text
06 – Hermes mit Home Assistant verbinden
```

Danach:

```text
07 – Hermes zu JARVIS machen: Persona, SOUL und erste Skills
08 – Kontrollierte JARVIS-/Home-Assistant-Tests
```
