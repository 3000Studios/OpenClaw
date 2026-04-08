# OpenClaw - Local AI Gateway

OpenClaw is a self-hosted AI gateway that connects your local Ollama models to external services and dashboards. This repo contains setup instructions, config templates, and troubleshooting guides for running OpenClaw on Windows with Cloudflare Tunnel for remote access.

---

## 🖥️ System Requirements

- Windows 10/11 (64-bit)
- Node.js v18+ (v24 recommended)
- npm v9+
- [Ollama](https://ollama.com) installed and running
- PowerShell (run as Administrator for setup)

---

## ⚡ Quick Install (PowerShell as Administrator)

### Step 1 — Clean any broken install

```powershell
cd $env:APPDATA
pm
ode_modules

taskkill /F /IM node.exe 2>$null
taskkill /F /IM openclaw.exe 2>$null
Start-Sleep -Seconds 2

Remove-Item -Recurse -Force openclaw -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force .openclaw-* -ErrorAction SilentlyContinue
```

### Step 2 — Reinstall clean

```powershell
npm cache clean --force
npm install -g openclaw@latest
```

### Step 3 — Onboard and start

```powershell
openclaw onboard --install-daemon
openclaw gateway start
```

### Step 4 — Verify locally

Open your browser and go to: http://127.0.0.1:18789/

---

## 🔧 Configuration

Copy `config.template.yaml` to `config.yaml` and fill in your values.

Key settings:
- `gateway.port` — default is `18789`
- `ollama.host` — default is `http://127.0.0.1:11434`
- `plugins` — disable unused channels (e.g. Feishu/ZaloUser) to avoid 500 errors
- `auth.insecure` — set to `false` in production

### Disable Feishu/ZaloUser plugin (fixes HTTP 500 error)

In your `config.yaml`, find the plugins section and set:

```yaml
plugins:
zalouser:
  enabled: false
feishu:
  enabled: false
```

Then restart the gateway:

```powershell
openclaw gateway start
```

---

## 🌐 Remote Access via Cloudflare Tunnel

This setup uses `cloudflared` to expose your local OpenClaw gateway at `https://openclaw.myappai.net`.

### Install cloudflared

Download from: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/

### Run the tunnel

```powershell
cloudflared tunnel --url http://127.0.0.1:18789
```

Or use a named tunnel with your Cloudflare account for a persistent URL.

### Secrets needed (store in repo secrets or `.env`)

| Secret | Description |
|---|---|
| `CLOUDFLARE_ACCOUNT_ID` | Your Cloudflare account ID |
| `CLOUDFLARE_API_TOKEN` | API token with tunnel permissions |
| `CLOUDFLARE_MASTER_TOKEN` | Master API token |
| `CLOUDFLARE_ZONE_ID` | Zone ID for myappai.net |
| `GH_PAT` | GitHub Personal Access Token |
| `GH_TOKEN` | GitHub token for API access |
| `GH_REPO` | Target repo (e.g. 3000Studios/OpenClaw) |
| `GEMINI_API_KEY` | Gemini API key for AI features |

---

## 🤖 Ollama Models

OpenClaw uses Ollama as the local model backend. Recommended model:

```powershell
ollama pull llama3.2:3b
```

Verify Ollama is running:

```powershell
curl http://127.0.0.1:11434/api/tags
```

Set agent model in OpenClaw dashboard: `ollama/llama3.2:3b`

---

## 🐛 Troubleshooting

### Gateway starts but port 18789 not listening

```powershell
# Check what's on the port
netstat -ano | findstr :18789

# Start with debug output
openclaw gateway start --debug
```

If another process owns the port, kill it:
```powershell
taskkill /F /PID <PID_NUMBER>
```

### NPM permission errors (EPERM)

Run PowerShell as Administrator. Windows file handles get stuck — the clean install steps above fix this.

### HTTP 500 errors

Usually caused by a missing module (`@larksuiteoapi/node-sdk`). Fix: disable the Feishu/ZaloUser plugin in config (see above).

### Gateway process exists but not accepting connections

The process is running but not binding to the port. Run with `--debug` to see the startup error:

```powershell
openclaw gateway start --debug
```

### Reinstall loop / unstable install

1. Kill all node processes
2. Delete openclaw from `$env:APPDATA
pm
ode_modules`
3. Run `npm cache clean --force`
4. Reinstall fresh

---

## 📁 Repo Structure

```
OpenClaw/
├── README.md              # This file
├── config.template.yaml   # Config template (copy to config.yaml)
├── setup.ps1              # Automated setup script
└── TROUBLESHOOTING.md     # Extended troubleshooting guide
```

---

## 🔗 Links

- Ollama: https://ollama.com
- Cloudflare Tunnel docs: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/
- OpenClaw dashboard (local): http://127.0.0.1:18789/
- OpenClaw dashboard (remote): https://openclaw.myappai.net
