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
│  WIS (CF Worker)    │
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
- **WIS** ([original repo](https://github.com/toverainc/willow-inference-server)): Originally Docker-based inference requiring a GPU. Tovera's hosted instance (`infer.tovera.io`) is dead. We replaced it with a [Cloudflare Worker](https://github.com/strugglingcomic/willow-cloudflare-wis) using Whisper (ASR) and Deepgram Aura-2 (TTS) at `willow-wis.strugglingcomic.workers.dev`. Endpoints require `?key=` auth.
- **Command Endpoint**: The HTTP endpoint Willow sends transcribed text to. This is where our Claude integration plugs in.

## Data Flow (high-level)

1. User says wake word → device starts listening
2. Audio streamed to WIS for speech-to-text
3. Transcribed text sent to configured command endpoint
4. Endpoint processes command and returns response text
5. Response sent to WIS for TTS
6. Audio played back through device speaker

> The exact API contract for the command endpoint (request/response format) should be verified against the [Willow docs](https://heywillow.io/) and WAS/WIS source code when we reach the integration phase.

## WIS Replacement Details

The Cloudflare Worker translates between Willow's native format and Cloudflare Workers AI:

| Direction | Willow sends | Worker does | Model |
|---|---|---|---|
| ASR | POST raw PCM (16kHz/16-bit/mono) with `x-audio-*` headers | Wraps PCM in WAV header, base64 encodes, calls Whisper | `@cf/openai/whisper-large-v3-turbo` |
| TTS | GET with `?text=` query param | Calls Aura-2, returns WAV stream | `@cf/deepgram/aura-2-en` |

Free tier: 10,000 neurons/day (~200 ASR + ~50 TTS requests).
