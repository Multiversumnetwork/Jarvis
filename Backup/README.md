# Backup

Automatische Home-Assistant-Backups der Instanz **JarvisMansion** (192.168.188.113).

- **Quelle:** HA-Backup-Zeitplan, täglich 03:00 Uhr (lokal auf der VM)
- **Sync:** Cron auf JarvisCore lädt das jeweils neueste automatische Backup
  herunter und committet es hierher (Skript: `~/.hermes/scripts/jarvis-ha-backup-git.py`)
- **Rotation:** die 7 neuesten `ha_backup_*.tar` bleiben im Ordner
- **Verschlüsselung:** Die Archive sind mit dem HA-Backup-Schlüssel verschlüsselt.
  Der Schlüssel liegt NICHT im Repo (JarvisCore: `~/.config/homeassistant/backup_encryption_key.txt`
  sowie in Marcs Passwortmanager). Ohne Schlüssel sind die Archive wertlos.

**Wiederherstellung:** HA → Einstellungen → System → Backups → Backup hochladen,
Schlüssel eingeben. Bei Totalausfall: frische HA-OS-VM aufsetzen, im Onboarding
„Aus Backup wiederherstellen" wählen.
