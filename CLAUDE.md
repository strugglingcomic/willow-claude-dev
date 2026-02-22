# willow-claude-dev

Tracking repo for building a Willow-based voice assistant with Claude integration on ESP32-S3-BOX-3.

## Willow Architecture

```
ESP32-S3-BOX-3 (firmware)
    ↕ HTTP/WebSocket
Willow Application Server (WAS) — device management, config
    ↕
Willow Inference Server (WIS) — ASR, TTS (see note below)
    ↕
Command Endpoint — where recognized speech goes for processing
```

Claude integration point: custom command endpoint middleware that receives transcribed speech and routes to Claude API.

**WIS replacement:** Tovera's hosted WIS (`infer.tovera.io`) is dead (DNS NXDOMAIN).
We run a Cloudflare Worker replacement at `willow-cloudflare-wis/` that provides
`/api/willow` (Whisper ASR) and `/api/tts` (Deepgram Aura-2 TTS). Endpoints require
`?key=` param — key stored in `pass willow-wis/api-key`. Free tier: ~200 ASR + ~50 TTS/day.

## WAS Config API

- GET config: `curl -s "http://localhost:8502/api/config?type=config"`
- POST config: `curl -X POST "http://localhost:8502/api/config?type=config&apply=true" -H "Content-Type: application/json" -d '{...}'`
- `apply=true` pushes config to the device immediately
- WIS URL fields: `wis_url`, `wis_tts_url`

## Willow Dev Workflow

Firmware builds use `./utils.sh` from the willow repo root. See the
[willow README](https://github.com/toverainc/willow#readme) for current subcommands and setup steps.

Server components (WAS, WIS) run via Docker — see their respective READMEs for docker compose instructions.

## Key Repos and Docs

- Firmware: https://github.com/toverainc/willow
- WAS: https://github.com/toverainc/willow-application-server
- WIS: https://github.com/toverainc/willow-inference-server
- Docs: https://heywillow.io/

**Always check these repos for current setup instructions, API formats, and configuration options.** This tracking repo captures our project-specific decisions and integration work, not a copy of upstream docs.

## Hardware

- Board: ESP32-S3-BOX-3 ([Espressif docs](https://github.com/espressif/esp-box))
- Connection: USB-C for flashing and serial monitor
