# willow-claude-dev

Tracking repo for building a Willow-based voice assistant with Claude integration on ESP32-S3-BOX-3.

## Willow Architecture

```
ESP32-S3-BOX-3 (firmware)
    ↕ HTTP/WebSocket
Willow Application Server (WAS) — device management, config (port 8502)
    ↕
Willow Inference Server (WIS) — ASR (Whisper), TTS, optional LLM (NVIDIA GPU required)
    ↕
Command Endpoint — where recognized speech goes for processing
```

Claude integration point: custom command endpoint middleware that receives transcribed speech from WIS and routes to Claude API.

## Willow Dev Workflow

All commands run from the willow repo root using `./utils.sh`:

```bash
# First-time setup
./utils.sh build-docker    # Build the ESP-IDF Docker container
./utils.sh install         # Install dependencies inside container
./utils.sh config          # Interactive menuconfig (set WiFi, WAS URL, etc.)

# Build and flash cycle
./utils.sh build           # Compile firmware
./utils.sh flash           # Flash to connected ESP32-S3-BOX-3
./utils.sh monitor         # Serial monitor for logs

# Server components (separate repos)
docker compose up          # Start WAS and/or WIS
```

## Key Repos

- Firmware: https://github.com/toverainc/willow
- WAS: https://github.com/toverainc/willow-application-server
- WIS: https://github.com/toverainc/willow-inference-server
- Docs: https://heywillow.io/

## Hardware

- Board: ESP32-S3-BOX-3
- Connection: USB-C for flashing and serial monitor
- WIS requires NVIDIA GPU with CUDA for local inference
