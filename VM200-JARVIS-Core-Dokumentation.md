# VM 200 – JARVIS Core

Stand: 26. August 2026

## Ergebnis

VM 200 wurde vollständig neu mit Debian 13 installiert und als JARVIS Core eingerichtet. Hermes läuft nativ auf Debian, ohne Docker. Der öffentliche Webchat ist unter `https://chat.jarvis.multiversum.network` über Caddy, NetBird und Authentik erreichbar.

Der Endzustand wurde durch einen vollständigen VM-Reboot geprüft. Netzwerk, QEMU Guest Agent, NetBird, Hermes Gateway, Dashboard, HTTPS und Authentik kamen danach automatisch wieder online.

## Abgrenzung

- VM 100 (`jarvis-mansion`, HAOS) wurde nicht verändert.
- VM 300 (`jarvis-services`) wurde nach der Migration wieder gestoppt.
- VM 200 bleibt eingeschaltet und besitzt Proxmox-Autostart.
- Auf VM 200 ist kein Docker installiert.

## VM- und Betriebssystemdaten

| Einstellung | Wert |
|---|---|
| Proxmox-VM | 200 |
| Hostname | `jarvis-core` |
| Betriebssystem | Debian 13.6 |
| Kernel | 6.12.105 |
| CPU | 2 vCPU |
| Arbeitsspeicher | 4 GiB |
| Bootdisk | 32 GiB |
| LAN-IP | `192.168.188.114` |
| NetBird-IP | `100.81.4.182` |
| Zeitzone | `Europe/Luxembourg` |
| Proxmox-Autostart | aktiviert |
| QEMU Guest Agent | aktiv |

Die LAN-IP ist in der FritzBox fest zugeordnet. Die öffentliche IPv4 des Heimanschlusses (`109.89.105.90`) wird für den Webchat nicht direkt veröffentlicht; der Zugriff läuft über den zentralen Hetzner-Server.

## Benutzer und SSH

- Administrativer Benutzer: `jarvis`
- `jarvis` ist unprivilegiert und verwendet `sudo` nur bei Bedarf.
- Direkter Root-Login bleibt deaktiviert; das Root-Konto ist gesperrt.
- Der frühere Benutzer `Marc` wurde entfernt.
- SSH akzeptiert ausschließlich Schlüsselanmeldung.
- Verwendeter 1Password-Schlüssel: `JARVIS VMs SSH 2026`
- Fingerprint: `SHA256:zJV0Oldp34Brh7q+KMQ0SXTPDllkbgGTyfnA0g7JdFU`

Der während der Installation verwendete temporäre Proxmox-SSH-Tunnel `vm200-ssh-portal.service` wurde beendet und als `inactive` bestätigt.

## Hermes

| Einstellung | Wert |
|---|---|
| Installation | nativ unter `/home/jarvis/.hermes/hermes-agent` |
| Version | 0.20.5 |
| Gateway-Dienst | `hermes-gateway.service` |
| Dashboard-Dienst | `hermes-dashboard.service` |
| Dashboard-Bindung | `100.81.4.182:9119` |
| Anbieter | Nous Portal |
| Standardmodell | `deepseek/deepseek-v4-pro` |

Beide Hermes-Dienste sind als systemd-Dienste eingerichtet und aktiviert. Konfiguration und Nous-Anmeldung liegen im Home-Verzeichnis des dedizierten Benutzers `jarvis` und überstehen Neustarts.

Funktionstests:

- CLI-Antwort über Nous: `JARVIS CORE OK`
- öffentlicher Webchat über Nous: `JARVIS WEBCHAT OK`
- Dashboard nach VM-Reboot: Gateway-Status `Running`

## Öffentlicher Zugriff

Zugriffsweg:

```text
Browser
  -> chat.jarvis.multiversum.network
  -> Hetzner Caddy (128.140.127.114)
  -> NetBird-Container + TCP-Brücke auf dem Hetzner-Server
  -> NetBird-Richtlinie zu VM 200
  -> Hermes Dashboard auf 100.81.4.182:9119
  -> Authentik-OIDC-Anmeldung
```

### DNS

- Zone: `multiversum.network`
- Record: `chat.jarvis`
- Typ: A
- Ziel: `128.140.127.114`
- TTL: 10 Minuten
- Es wurde bewusst kein AAAA-Record angelegt.

### TLS und Caddy

Caddy stellt das Zertifikat automatisch über Let's Encrypt aus. Der produktive Caddy-Block lautet:

```caddyfile
chat.jarvis.multiversum.network {
    reverse_proxy netbird:19119
}
```

Vor der Änderung wurde eine Sicherung als `/opt/multiversum/caddy/Caddyfile.bak-20260826-1212z` angelegt. Die Konfiguration wurde validiert und geladen. Der temporäre Schreibzugriff des NetBird-Containers auf `/opt/multiversum/caddy` wurde nach Abschluss wieder entfernt.

### NetBird

| Komponente | Wert |
|---|---|
| Hetzner-Peer | `multiversum-services` / `100.81.233.149` |
| VM-200-Peer | `jarvis-core` / `100.81.4.182` |
| Quellgruppe | `JARVIS Proxy` |
| Zielgruppe | `JARVIS Core` |
| Freigabe | TCP 9119, nur Proxy → Core |

Auf dem Hetzner-Server läuft der Portainer-Stack `netbird` mit:

- `netbirdio/netbird:0.77.1`
- `alpine/socat:1.8.1.3` als `jarvis-dashboard-bridge`
- interner Brückenport `19119`
- Ziel `100.81.4.182:9119`

Der einmalige NetBird-Setup-Key wurde nach erfolgreicher Registrierung aus dem Stack entfernt.

### Authentik

| Einstellung | Wert |
|---|---|
| Anwendung | `JARVIS Chat` |
| Slug | `jarvis-chat` |
| Gruppe | `JARVIS` |
| Verfahren | OAuth2/OIDC, Authorization Code + PKCE |
| Client-Typ | Public |
| Scopes | `openid profile email` |
| Redirect URI | `https://chat.jarvis.multiversum.network/auth/callback` |
| Issuer | `https://login.multiversum.network/application/o/jarvis-chat/` |
| Client-ID | `0UAQHFK3cHyAkPN60zqVX99aTokBeSUHXhpXA0LE` |

Es werden keine Passwörter, Nous-Tokens oder privaten Schlüssel in dieser Dokumentation gespeichert.

## Betriebsprüfung

Geprüft und erfolgreich:

- Debian 13 vollständig aktualisiert
- feste LAN-IP und Hostname
- Root gesperrt, SSH-Schlüsselpflicht aktiv
- QEMU Guest Agent
- Hermes Gateway und Dashboard
- Nous Portal mit `deepseek-v4-pro`
- HTTPS-Zertifikat und öffentliche Weiterleitung
- Authentik-Anmeldung inklusive PKCE-Callback
- NetBird-Richtlinie und TCP-Brücke
- VM-200-Autostart in Proxmox
- vollständiger VM-Reboot mit automatischer Wiederherstellung aller Dienste
- VM 300 wieder gestoppt
- temporäre Wartungszugriffe entfernt

## Wartung

Auf VM 200:

```bash
sudo systemctl status hermes-gateway hermes-dashboard netbird qemu-guest-agent
sudo journalctl -u hermes-gateway -u hermes-dashboard --since today
sudo apt update
apt list --upgradable
```

Öffentlichen Endpunkt prüfen:

```bash
curl -I https://chat.jarvis.multiversum.network/
```

Ohne gültige Browser-Sitzung ist eine Weiterleitung zur Hermes-/Authentik-Anmeldung (`HTTP 302`) das erwartete Ergebnis.

## Snapshots

- Snapshot des sauberen Debian-Grundsystems: `jarvis-core_debian13_clean_20260826`
- Frühere, nicht mehr passende VM-200-Snapshots wurden vor der Neuinstallation entfernt.
- Abschließender Snapshot des vollständigen JARVIS Core: `jarvis-core-complete-2026-08-26`
- Erstellt am 26. August 2026 um 15:38 Uhr über Proxmox.
- Snapshot ohne RAM-Abbild; das Gast-Dateisystem wurde über den QEMU Guest Agent eingefroren und anschließend wieder freigegeben.
- Proxmox meldete `TASK OK`; VM 200 blieb anschließend im Status `running`.

Beschreibung des finalen Snapshots:

```text
Debian 13.6, Hermes 0.20.5 nativ, Nous/DeepSeek v4 Pro, NetBird, Authentik-OIDC, Caddy-Publikation und Reboot-Test erfolgreich
```
