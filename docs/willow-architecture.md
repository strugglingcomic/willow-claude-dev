# Willow Architecture

> For authoritative details on each component, see the official repos and [Willow docs](https://heywillow.io/).

## Component Overview

```
┌─────────────────────┐
│  ESP32-S3-BOX-3     │
│  (Willow firmware)  │
└────────┬────────────┘
         │ HTTP / WebSocket
         ▼
┌─────────────────────┐
│  WAS                │
│  Device management  │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  WIS                │
│  ASR + TTS          │
└────────┬────────────┘
         │ Transcribed text
         ▼
┌─────────────────────┐
│  Command Endpoint   │
│  (Custom middleware) │
│  → Claude API       │
└─────────────────────┘
```

- **Firmware** ([repo](https://github.com/toverainc/willow)): Runs on the ESP32-S3-BOX-3. Handles wake word detection, audio capture, display, and speaker output.
- **WAS** ([repo](https://github.com/toverainc/willow-application-server)): Docker-based device management — config UI, OTA updates. See repo README for ports and setup.
- **WIS** ([repo](https://github.com/toverainc/willow-inference-server)): Docker-based inference — ASR (speech-to-text), TTS (text-to-speech), optional LLM. See repo README for GPU requirements.
- **Command Endpoint**: The HTTP endpoint Willow sends transcribed text to. This is where our Claude integration plugs in.

## Data Flow (high-level)

1. User says wake word → device starts listening
2. Audio streamed to WIS for speech-to-text
3. Transcribed text sent to configured command endpoint
4. Endpoint processes command and returns response text
5. Response sent to WIS for TTS
6. Audio played back through device speaker

> The exact API contract for the command endpoint (request/response format) should be verified against the [Willow docs](https://heywillow.io/) and WAS/WIS source code when we reach the integration phase.
