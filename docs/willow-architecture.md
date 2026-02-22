# Willow Architecture

## Component Overview

```
┌─────────────────────┐
│  ESP32-S3-BOX-3     │
│  (Willow firmware)  │
│  - Wake word detect │
│  - Audio capture    │
│  - LCD display      │
│  - Speaker output   │
└────────┬────────────┘
         │ HTTP / WebSocket
         ▼
┌─────────────────────┐
│  WAS (port 8502)    │
│  - Device mgmt UI   │
│  - OTA updates      │
│  - Config storage   │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  WIS                │
│  - Whisper ASR      │
│  - TTS engine       │
│  - Optional LLM     │
│  (Needs NVIDIA GPU) │
└────────┬────────────┘
         │ Transcribed text
         ▼
┌─────────────────────┐
│  Command Endpoint   │
│  (Custom middleware) │
│  - Receives text    │
│  - Routes to Claude │
│  - Returns response │
└─────────────────────┘
```

## Data Flow

1. User says wake word ("Hi Willow") → device starts listening
2. Audio streamed to WIS for speech-to-text (Whisper)
3. Transcribed text sent to configured command endpoint
4. Endpoint processes command and returns response text
5. Response sent to WIS for TTS
6. Audio played back through device speaker

## API Endpoints (WAS)

- `http://<host>:8502` — Web UI for device management
- Device configuration stored and pushed via WAS

## Command Endpoint Interface

The command endpoint receives POST requests with transcribed text and returns a response. This is where Claude integration would plug in.

Expected format (to be confirmed during setup):
```json
// Request
{"text": "what's the weather like today", "device_id": "..."}

// Response
{"speech": "It's currently 72 degrees and sunny."}
```
