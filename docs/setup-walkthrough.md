# Setup Walkthrough — Willow + WAS on Linux Mint / Ubuntu

Step-by-step record of what it actually took to get WAS running and flash the ESP32-S3-BOX-3. Done on Linux Mint 22.2 (Ubuntu noble base).

## 1. Install Docker Engine

Homebrew only installs the Docker CLI (no daemon), and `brew install --cask docker` fails on Linux Mint. Use Docker's official apt repo instead.

```bash
# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker apt repo (Mint 22 = Ubuntu noble)
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update -qq -o Dir::Etc::sourcelist=/etc/apt/sources.list.d/docker.list -o Dir::Etc::sourceparts="-"
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Fix: Docker group permissions

By default, Docker requires `sudo`. Add your user to the `docker` group:

```bash
sudo usermod -aG docker codywang
```

**You must log out and log back in** for this to take effect. `newgrp docker` works as a temporary workaround within the current shell, but Chrome and other apps won't pick it up until you do a full logout/login.

## 2. Clone repos

```bash
cd ~/workspace/HOME-AI_projects
git clone https://github.com/toverainc/willow.git
git clone https://github.com/toverainc/willow-application-server.git
```

## 3. Start WAS

```bash
docker run \
  --detach \
  --name=willow-application-server \
  --env TZ="America/New_York" \
  --pull=always \
  --network=host \
  --restart=unless-stopped \
  --volume=was-storage:/app/storage \
  ghcr.io/heywillow/willow-application-server
```

Verify it's running:

```bash
docker ps --filter name=willow-application-server
curl -s -o /dev/null -w "%{http_code}" http://localhost:8502
# → 307 (redirect to UI — normal)
```

**WAS Web UI:** http://localhost:8502

### Restarting WAS later

If the container exists but is stopped:

```bash
docker start willow-application-server
```

Alternatively, the repo has `./utils.sh run` but it defaults to a wrong image name. Either set the image explicitly or just use the `docker run` command above.

```bash
# If using utils.sh:
IMAGE=ghcr.io/heywillow/willow-application-server ./utils.sh run
```

## 4. Install Chrome

The Willow web flasher requires Chrome for Web Serial API support. Firefox does not support Web Serial.

```bash
cd /tmp
curl -fsSL -o google-chrome-stable.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable.deb
```

## 5. Flash the ESP32-S3-BOX-3

### Hardware setup

Plug in the ESP32-S3-BOX-3 via USB-C. Confirm it's detected:

```bash
lsusb | grep Espressif
# → Espressif USB JTAG/serial debug unit

ls /dev/ttyACM0
# → /dev/ttyACM0
```

### Fix: Serial port permissions

The serial device `/dev/ttyACM0` requires the `dialout` group:

```bash
sudo usermod -aG dialout codywang
```

**Again, log out and log back in** for Chrome to inherit the new group. This is required — `newgrp` alone is not enough for Chrome.

### Open the web flasher

The web flasher is accessed **through the WAS UI**, not by navigating to the flash URL directly. Direct navigation causes the page to call `window.close()` and immediately close.

1. Open Chrome and go to **http://localhost:8502**
2. In the WAS UI, click the **Flash** button/link
3. This opens the flash page as a popup: `https://flash.heywillow.io/?wasURL=ws://localhost:8502/ws&showPreReleases=false`
4. Select the serial port when Chrome prompts
5. Follow the on-screen flashing steps

If the flash page closes immediately when opened directly, that's expected — it must be launched from within WAS.

## 6. Replace WIS with Cloudflare Worker

Tovera's hosted WIS (`infer.tovera.io`) is dead (DNS NXDOMAIN) and no other public instances exist. Running WIS locally requires a GPU with ~6GB VRAM. Instead, we deployed a lightweight Cloudflare Worker that provides WIS-compatible endpoints using Cloudflare Workers AI.

### Create and deploy the Worker

The Worker lives in a separate repo: [willow-cloudflare-wis](https://github.com/strugglingcomic/willow-cloudflare-wis).

```bash
cd ~/workspace/HOME-AI_projects/willow-cloudflare-wis
npm install
npx wrangler login       # authenticate with Cloudflare account
npx wrangler deploy      # deploy to edge
# → https://willow-wis.strugglingcomic.workers.dev
```

### Models used

| Endpoint | Model | Cost |
|---|---|---|
| `/api/willow` (ASR) | `@cf/openai/whisper-large-v3-turbo` | ~$0.0005/audio min |
| `/api/tts?text=...` (TTS) | `@cf/deepgram/aura-2-en` (speaker: luna) | ~$0.03/1k chars |

Free tier: 10,000 neurons/day (~200 ASR + ~50 TTS requests).

### Set the API key

Endpoints require a `?key=` query parameter for authentication. Generate a key and store it as a Cloudflare Worker secret:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
# → copy the output

echo "YOUR_KEY_HERE" | npx wrangler secret put API_KEY
```

Store the key durably with `pass`:

```bash
echo "YOUR_KEY_HERE" | pass insert -e willow-wis/api-key
```

Retrieve later: `pass willow-wis/api-key`

### Update WAS config

Point WAS at the new Worker endpoints (with key in the URL):

```bash
KEY=$(pass willow-wis/api-key)
curl -X POST "http://localhost:8502/api/config?type=config&apply=true" \
  -H "Content-Type: application/json" \
  -d "{\"wis_url\": \"https://willow-wis.strugglingcomic.workers.dev/api/willow?key=${KEY}\", \"wis_tts_url\": \"https://willow-wis.strugglingcomic.workers.dev/api/tts?key=${KEY}\"}"
```

Verify:

```bash
curl -s "http://localhost:8502/api/config?type=config" | python3 -c "
import sys, json
d = json.load(sys.stdin)
for k, v in d.items():
    if 'wis' in k.lower():
        print(f'{k}: {v}')
"
```

### Monitoring

```bash
npx wrangler tail   # stream live Worker logs
```

## Gotchas / Lessons Learned

| Issue | Fix |
|---|---|
| `brew install docker` only installs CLI, no daemon | Use Docker's official apt repo instead |
| `brew install --cask docker` fails on Mint | Same — use apt |
| `sudo usermod -aG docker $USER` fails (empty $USER in sudo) | Use explicit username: `sudo usermod -aG docker codywang` |
| Docker commands need `sudo` after usermod | Must logout/login for group to take effect |
| Web flasher page closes immediately | Must open from within WAS UI, not directly |
| Serial port "permission denied" in Chrome | Add user to `dialout` group + logout/login |
| `utils.sh run` can't find image | Set `IMAGE=ghcr.io/heywillow/willow-application-server` or use `docker run` directly |
| `infer.tovera.io` DNS NXDOMAIN — WIS is dead | Replaced with Cloudflare Worker (see step 6) |
| WAS config POST requires `apply=true` query param | Without it, config saves but doesn't push to device |
| GitHub HTTPS push fails from this machine | Use SSH remotes: `git remote set-url origin git@github.com:...` |
| Cloudflare Worker secrets are write-only | Can't read back the value — store keys in `pass` |
