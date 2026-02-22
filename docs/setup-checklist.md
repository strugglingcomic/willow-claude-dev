# Setup Checklist

Each component's setup instructions live in its own repo. This checklist tracks *our* progress through them.

## Prerequisites

- [ ] ESP32-S3-BOX-3 hardware connected via USB-C
- [ ] Docker installed and running
- [ ] GPU setup verified (if running WIS locally — see [WIS repo](https://github.com/toverainc/willow-inference-server) for requirements)

## Firmware

Follow the [willow repo README](https://github.com/toverainc/willow#readme) for build/flash instructions.

- [ ] Clone repo
- [ ] Build Docker dev container
- [ ] Configure (WiFi, server URLs)
- [ ] Build and flash firmware
- [ ] Verify device boots and connects (serial monitor)

## WAS

Follow the [WAS repo README](https://github.com/toverainc/willow-application-server#readme).

- [ ] Clone repo
- [ ] Start via Docker
- [ ] Access management UI
- [ ] Device appears and is manageable

## WIS

Follow the [WIS repo README](https://github.com/toverainc/willow-inference-server#readme).

- [ ] Clone repo
- [ ] Verify GPU access in Docker
- [ ] Start via Docker
- [ ] Test speech recognition with sample audio

## Integration (our custom work)

- [ ] Build custom command endpoint (Python/FastAPI)
- [ ] Configure Willow to send commands to our endpoint
- [ ] Test end-to-end: speak → transcribe → Claude → TTS → playback
