# Setup Checklist

## Prerequisites

- [ ] ESP32-S3-BOX-3 hardware connected via USB-C
- [ ] Docker installed and running
- [ ] NVIDIA GPU + CUDA drivers (for WIS local inference)
- [ ] `esptool.py` or ESP-IDF toolchain available

## Firmware Setup

- [ ] Clone willow repo: `git clone https://github.com/toverainc/willow.git`
- [ ] Build Docker container: `./utils.sh build-docker`
- [ ] Install dependencies: `./utils.sh install`
- [ ] Configure: `./utils.sh config` (set WiFi SSID/password, WAS URL)
- [ ] Build firmware: `./utils.sh build`
- [ ] Flash to device: `./utils.sh flash`
- [ ] Verify with serial monitor: `./utils.sh monitor`

## WAS Setup

- [ ] Clone WAS repo: `git clone https://github.com/toverainc/willow-application-server.git`
- [ ] Start with Docker: `docker compose up`
- [ ] Access UI at http://localhost:8502
- [ ] Device appears in management interface

## WIS Setup

- [ ] Clone WIS repo: `git clone https://github.com/toverainc/willow-inference-server.git`
- [ ] Verify NVIDIA GPU access in Docker (`nvidia-smi`)
- [ ] Start with Docker: `docker compose up`
- [ ] Test ASR endpoint with sample audio

## Integration

- [ ] Build custom command endpoint (Python/FastAPI)
- [ ] Configure Willow to use custom endpoint
- [ ] Test end-to-end: speak → transcribe → Claude → TTS → playback
