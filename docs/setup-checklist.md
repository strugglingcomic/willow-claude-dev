# Setup Checklist

Each component's setup instructions live in its own repo. This checklist tracks *our* progress through them.

For detailed step-by-step commands, see [setup-walkthrough.md](setup-walkthrough.md).

## Prerequisites

- [x] ESP32-S3-BOX-3 hardware connected via USB-C
- [x] Docker installed and running (via apt, not brew — see walkthrough)
- [x] User added to `docker` group
- [x] User added to `dialout` group (for serial port access)
- [x] Chrome installed (required for Web Serial API)
- [x] ~~GPU setup verified~~ — Not needed; using Cloudflare Worker instead of local WIS

## Firmware

- [x] Clone repo
- [x] Configure (WiFi, server URLs — WIS URLs updated to Cloudflare Worker)
- [x] Flash via WAS web flasher (Chrome → http://localhost:8502 → Flash)
- [ ] Verify device boots and connects (serial monitor)

## WAS

- [x] Clone repo
- [x] Start via Docker (`docker run` with `ghcr.io/heywillow/willow-application-server`)
- [x] Access management UI at http://localhost:8502
- [ ] Device appears and is manageable

## WIS (Cloudflare Worker replacement)

Tovera's hosted WIS (`infer.tovera.io`) is dead. No local GPU available. Replaced with a Cloudflare Worker.
See [setup-walkthrough.md § 6](setup-walkthrough.md#6-replace-wis-with-cloudflare-worker) for details.

- [x] Deploy Cloudflare Worker (`willow-cloudflare-wis/`)
- [x] Set API key as Worker secret + store in `pass willow-wis/api-key`
- [x] Update WAS config with new WIS URLs (with `?key=` param)
- [x] Verify health endpoint responds
- [x] Verify TTS endpoint returns valid WAV audio
- [ ] Test ASR end-to-end with device (speak → transcribe)

## Integration (our custom work)

- [ ] Build custom command endpoint (Python/FastAPI)
- [ ] Configure Willow to send commands to our endpoint
- [ ] Test end-to-end: speak → transcribe → Claude → TTS → playback
