<div align="center">

# FreeDeepseekAPI

**Local OpenAI-compatible API proxy for DeepSeek Web Chat**

[![Node.js](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg)](https://nodejs.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-100%25%20passing-success)](tests/)
[![DeepSeek Web](https://img.shields.io/badge/DeepSeek%20Web-Unified%20Mode-blue)](https://chat.deepseek.com)

[Quick Start](#-quick-start) •
[Features](#-features) •
[Daemon Mode](#-daemon-mode) •
[API Examples](#-api-examples) •
[Models](#-models) •
[Multi-Account Pool](#-multi-account-pool) •
[Endpoints](#-endpoints)

</div>

---

FreeDeepseekAPI runs a local API proxy server for **DeepSeek Web Chat** (`chat.deepseek.com`), allowing you to connect DeepSeek Web to Open WebUI, LiteLLM, Hermes, Claude Code, OpenAI SDK clients, and any other OpenAI-compatible applications.

The proxy connects using your logged-in DeepSeek web session via a separate Chrome profile. It accepts standard OpenAI/Anthropic/Responses requests, forwards them to DeepSeek Web, and returns standard completions or SSE streams.

> ⚠️ This is an experimental web-chat proxy. DeepSeek may modify internal web API protocols without notice. For production reliability, use the official paid DeepSeek API.

---

## Table of Contents
- [What It Provides](#-what-it-provides)
- [Features](#-features)
- [Quick Start](#-quick-start)
- [Daemon Mode & Terminal Controls](#-daemon-mode--terminal-controls)
- [Windows Guide](#-windows-guide)
- [Linux & Chromium Guide](#-linux--chromium-guide)
- [VPS & Headless Setup](#-vps--headless-setup)
- [Permanent VPS Service (systemd)](#-permanent-vps-service-systemd)
- [Session Reuse & Reset](#-session-reuse--reset)
- [Multi-Account Pool](#-multi-account-pool)
- [Health Check & Doctor](#-health-check--doctor)
- [API Examples](#-api-examples)
- [Models](#-models)
- [Endpoints](#-endpoints)
- [Open WebUI Integration](#-open-webui-integration)
- [Testing](#-testing)

---

## ✨ What It Provides
- Use DeepSeek Web as a local API endpoint.
- Connect DeepSeek to Open WebUI and any OpenAI-compatible client.
- Receive standard JSON responses or live SSE streaming.
- Use reasoning models (R1) with dedicated `reasoning_content`.
- Use Anthropic Messages API shim for Claude Code / Anthropic SDK.
- Use OpenAI Responses API shim for Codex/OpenAI agent clients.
- Isolate independent web sessions per agent/user.
- Detach to background daemon mode with a single keypress (`Ctrl+D`).

---

## 🚀 Features
- **Streaming:** SSE chunks and standard non-stream JSON responses.
- **Reasoning Output:** Dedicated `reasoning_content` for thinking models.
- **Tool Calling:** Parses OpenAI tools, Anthropic tools, and Responses function tools.
- **Model Capabilities:** `GET /v1/model-capabilities` with alias-to-web-mode mapping.
- **Agent Sessions:** Distinct DeepSeek session per `user` or agent id.
- **Session Recovery:** Automatic recovery and reset of stale conversation chains.
- **Multi-Account Pool:** Automatic rotation, load-balancing, and cooldown on rate limits.
- **Image Uploads:** Upload and analyze images via `deepseek-vision`.
- **Zero External Dependencies:** Built with native Node.js libraries. Node.js 18+ for proxy, Node.js 22+ for browser auth.

---

## ⚡ Quick Start

### 1. Authenticate with DeepSeek Web
Run the auth tool to log in via Chrome:
```bash
npm run auth
```
1. Select option `1` (or run `npm run auth -- --login`).
2. Log into DeepSeek in the opened Chrome window.
3. Send a short test message (e.g. `ok`) in DeepSeek chat.
4. Return to your terminal and press `Enter`. Credentials are saved to `deepseek-auth.json`.

### 2. Start the API Proxy
Run:
```bash
npm start
```
The server starts immediately without any blocking prompts:
```
[DS-API] Server on http://127.0.0.1:9655 (multi-agent sessions enabled)
[DS-API] POST /v1/chat/completions (OpenAI Chat Completions)
...
[DS-API] 💡 Press Ctrl+D to send to daemon mode (background) | Press Ctrl+C to stop
```

---

## 😈 Daemon Mode & Terminal Controls

FreeDeepseekAPI is designed to be completely terminal-friendly:

### 1. Interactive Keybindings (Foreground Mode)
While `npm start` is running in your terminal:
- **`Ctrl + D`**: Detaches the process immediately and runs it in the background as a daemon! Your terminal prompt is freed, and server logs are redirected to `server.log`.
- **`Ctrl + C`**: Cleanly shuts down the server.

### 2. Start Directly in Daemon Mode
To start the proxy directly in the background without keeping the terminal open:
```bash
npm run daemon
# or: node server.js --daemon
```

### 3. Stop the Daemon
To stop any active background instance:
```bash
npm run stop
# or: node server.js --stop
```

### 4. Interactive Menu (Optional)
If you want the old interactive menu:
```bash
npm start -- --menu
# or: MENU=1 npm start
```

---

## 🪟 Windows Guide

### PowerShell
```powershell
# Authenticate
npm run auth

# Start server
npm start

# Or start as daemon
npm run daemon
```

If Chrome is not found:
```powershell
$env:CHROME_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe"
npm run auth
```

---

## 🐧 Linux & Chromium Guide

Chrome and Chromium are automatically detected at standard locations (`/usr/bin/google-chrome`, `/usr/bin/google-chrome-stable`, `/usr/bin/chromium`, `/usr/bin/chromium-browser`, `/snap/bin/chromium`).

```bash
# Authenticate
npm run auth

# Start
npm start
```

To run in background:
Press `Ctrl+D` while running, or run:
```bash
npm run daemon
```

---

## 🖥 VPS & Headless Setup

For headless servers without a GUI display:

1. **On your local computer:**
   ```bash
   npm run auth
   ```
2. **Copy `deepseek-auth.json` to the VPS:**
   ```bash
   scp ./deepseek-auth.json user@vps-ip:/opt/FreeDeepseekAPI/
   ```
3. **On your VPS:**
   ```bash
   npm run auth:import -- --input ./deepseek-auth.json
   npm run daemon
   ```

---

## 🛠 Permanent VPS Service (systemd)

A hardened systemd unit file is provided in `deploy/free-deepseek-api.service`.

```bash
# 1. Create dedicated user and copy directory
sudo useradd -r -s /usr/sbin/nologin -d /opt/FreeDeepseekAPI freedeepseek
sudo cp -r . /opt/FreeDeepseekAPI
sudo chown -R freedeepseek:freedeepseek /opt/FreeDeepseekAPI

# 2. Configure auth files
sudo mkdir -p /etc/free-deepseek-api
sudo cp deepseek-auth.json /etc/free-deepseek-api/
echo "your-secret-api-key" | sudo tee /etc/free-deepseek-api/proxy-api-key
sudo chown -R freedeepseek:freedeepseek /etc/free-deepseek-api
sudo chmod 600 /etc/free-deepseek-api/*

# 3. Enable and start service
sudo cp deploy/free-deepseek-api.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now free-deepseek-api

# 4. Check status
sudo systemctl status free-deepseek-api
sudo journalctl -u free-deepseek-api -f
```

---


---

## 🦭 Rootless Podman

The container image is intended for non-interactive proxy execution only. Perform browser login on the host with `npm run auth`: auth scripts and `deepseek-auth.json` are never copied into the container image.

Run Podman as an unprivileged user without `sudo`.

1. Build the local image:
```bash
podman build --format docker --tag localhost/free-deepseek-api:local --file Containerfile .
```

2. Pass DeepSeek auth and a separate proxy API key via Podman secrets:
```bash
podman secret create --replace free-deepseek-auth ./deepseek-auth.json

printf 'Proxy API key: '
IFS= read -r -s PROXY_API_KEY
printf '\n'
printf '%s' "$PROXY_API_KEY" |
  podman secret create --replace free-deepseek-proxy-key -
```

3. Run the container with least privileges:
```bash
podman run --detach \
  --name free-deepseek-api \
  --publish 127.0.0.1:9655:9655 \
  --secret free-deepseek-auth,target=deepseek-auth.json,uid=1000,gid=1000,mode=0400 \
  --secret free-deepseek-proxy-key,target=proxy-api-key,uid=1000,gid=1000,mode=0400 \
  --read-only \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  localhost/free-deepseek-api:local
```

4. Verify health and readiness:
```bash
podman healthcheck run free-deepseek-api
curl --fail http://127.0.0.1:9655/readyz
curl --fail -H "Authorization: Bearer $PROXY_API_KEY" http://127.0.0.1:9655/v1/models
```

Stop and clean up container and secrets:
```bash
podman stop free-deepseek-api
podman rm free-deepseek-api
podman secret rm free-deepseek-auth free-deepseek-proxy-key
unset PROXY_API_KEY
```

## 🩺 Diagnostics / Doctor
Run diagnostic checks on your environment and authentication:
```bash
npm run doctor
```
Or test network connectivity with live ping:
```bash
npm run doctor -- --live
```

---

## 🧪 API Examples

### Chat Completions (OpenAI Compatible)
```bash
curl http://127.0.0.1:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [
      {"role": "user", "content": "Hello! What model are you?"}
    ]
  }'
```

### Reasoning Mode (DeepSeek R1)
```bash
curl http://127.0.0.1:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-reasoner",
    "messages": [
      {"role": "user", "content": "How many r'''s are in strawberry?"}
    ]
  }'
```

### Streaming (SSE)
```bash
curl http://127.0.0.1:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "stream": true,
    "messages": [
      {"role": "user", "content": "Count from 1 to 5."}
    ]
  }'
```

### Anthropic Messages API (Claude Code / Anthropic SDK)
```bash
curl http://127.0.0.1:9655/v1/messages \
  -H "Content-Type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "deepseek-chat",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "Hello via Anthropic Messages shim!"}
    ]
  }'
```

---

## 🧠 Models

| Model ID | Web Backend Mode | Description |
| :--- | :--- | :--- |
| `deepseek-chat` | Unified Mode | Standard chat completions (fast, default) |
| `deepseek-reasoner` | Unified + DeepThink | Reasoning mode (R1) with `reasoning_content` |
| `deepseek-r1` | Unified + DeepThink | Alias for `deepseek-reasoner` |
| `deepseek-chat-search` | Unified + Web Search | Chat with real-time web search enabled |
| `deepseek-reasoner-search` | Unified + DeepThink + Search | Reasoning with web search |
| `deepseek-vision` | Unified + Vision | Multimodal image understanding |

Check real-time capabilities with:
```bash
curl http://127.0.0.1:9655/v1/model-capabilities
```

---

## 🔌 Endpoints

- `POST /v1/chat/completions` — OpenAI Chat Completions (streaming & non-streaming)
- `POST /v1/messages` — Anthropic Messages shim for Claude Code
- `POST /v1/responses` — OpenAI Responses API shim
- `GET /v1/models` — List of supported OpenAI models
- `GET /v1/model-capabilities` — Real model capabilities and mapping
- `GET /v1/sessions` — Active multi-agent sessions
- `POST /reset-session?agent=<id>` — Reset session for a specific agent
- `POST /reset-session?agent=all` — Reset all active sessions

---

## 🖥 Open WebUI Integration

Add FreeDeepseekAPI to Open WebUI:
1. Open **Admin Settings → Connections → OpenAI API**.
2. Add connection:
   - **URL:** `http://127.0.0.1:9655/v1`
   - **Key:** `sk-anything` (or any dummy text)
3. Save. All models (`deepseek-chat`, `deepseek-reasoner`, etc.) will appear automatically.

---

## 🧪 Testing

Run complete test suite (unit, integration, vision tests):
```bash
npm test
```

---

## 📄 License
MIT License. Created by [ForgetMeAI](https://t.me/forgetmeai).
