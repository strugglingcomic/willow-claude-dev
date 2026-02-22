# willow-claude-dev

Exploring [Willow](https://heywillow.io/) as a voice assistant platform with Claude integration, running on an ESP32-S3-BOX-3.

## What is this?

A tracking and documentation repo for getting a working Willow development environment set up and integrating Claude as the "brain" behind voice commands. Similar to a lab notebook — documenting setup steps, architecture decisions, and integration experiments.

## Hardware

- **[ESP32-S3-BOX-3](https://github.com/espressif/esp-box)** — Espressif's AI development kit with microphone array, speaker, and LCD display

## Willow Components

| Component | Repo | Description |
|-----------|------|-------------|
| Willow firmware | [toverainc/willow](https://github.com/toverainc/willow) | C/ESP-IDF firmware for the device |
| WAS | [toverainc/willow-application-server](https://github.com/toverainc/willow-application-server) | Device management and config UI (Docker) |
| WIS | [toverainc/willow-inference-server](https://github.com/toverainc/willow-inference-server) | Speech recognition, TTS (Docker, GPU) |

See each repo's README for current setup instructions, ports, and requirements.

## Goals

1. Get Willow firmware building and flashing to ESP32-S3-BOX-3
2. Run WAS locally for device management
3. Set up WIS for local speech recognition
4. Build a custom command endpoint that routes to Claude API
5. Have a working voice assistant: speak → Willow transcribes → Claude responds → TTS plays back

## Current Status

**Phase: Initial setup** — repo created, hardware in hand, documenting architecture and setup steps.

## Docs

- [Architecture overview](docs/willow-architecture.md)
- [Setup checklist](docs/setup-checklist.md)
- [Claude integration ideas](docs/claude-integration-ideas.md)

## Links

- [Willow documentation](https://heywillow.io/)
- [ESP32-S3-BOX-3 docs](https://github.com/espressif/esp-box)
