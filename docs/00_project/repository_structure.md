# Repository-Struktur

Empfohlene Struktur für das JARVIS-Repository:

```text
Jarvis/
├── README.md
├── docs/
│   ├── 00_project/
│   ├── 01_architecture/
│   ├── 02_hardware/
│   ├── 03_home_assistant/
│   ├── 04_voice_llm/
│   ├── 05_apple_home/
│   ├── 06_scenes/
│   ├── 07_inventory/
│   ├── 08_garden/
│   ├── 09_cameras/
│   ├── 10_drones/
│   ├── 11_security/
│   └── 12_changelog/
├── home-assistant/
│   ├── automations/
│   ├── scripts/
│   ├── scenes/
│   ├── dashboards/
│   ├── packages/
│   └── blueprints/
└── assets/
    ├── images/
    └── diagrams/
```

## Spätere Erweiterung

Wenn Home Assistant Konfigurationen versioniert werden:

```text
home-assistant/
├── automations/
├── scripts/
├── scenes/
├── helpers/
├── dashboards/
├── packages/
└── blueprints/
```

## Regel

Dokumentation zuerst, produktive Konfiguration später.
