# FreeDeepseekAPI

> 🇬🇧 **Translation-only fork** of [ForgetMeAI/FreeDeepseekAPI](https://github.com/ForgetMeAI/FreeDeepseekAPI). All code remains the original author's work — this fork only translates the UI text and documentation into English. **Huge thanks to [ForgetMeAI](https://t.me/forgetmeai) for the original project. Please support the original author, not this fork:** https://github.com/ForgetMeAI/FreeDeepseekAPI

<p align="center">
  <strong>Local OpenAI-compatible API proxy for DeepSeek Web Chat</strong>
</p>

<p align="center">
  <a href="https://github.com/ForgetMeAI/FreeDeepseekAPI/blob/main/LICENSE"><img alt="License MIT" src="https://img.shields.io/badge/license-MIT-green.svg" /></a>
  <img alt="Node.js 18 plus" src="https://img.shields.io/badge/node-18%2B-339933.svg" />
  <img alt="No npm dependencies" src="https://img.shields.io/badge/dependencies-0-blue.svg" />
  <img alt="OpenAI compatible" src="https://img.shields.io/badge/OpenAI-compatible-111111.svg" />
</p>

<p align="center">
  <a href="#-quick-start">Quick start</a> •
  <a href="#-features">Features</a> •
  <a href="#-request-examples">Examples</a> •
  <a href="#-models">Models</a> •
  <a href="#-endpoints">Endpoints</a> •
  <a href="#-open-webui">Open WebUI</a>
</p>

FreeDeepseekAPI runs a local API server for **DeepSeek Web Chat** (`chat.deepseek.com`) and lets you connect DeepSeek Web to Open WebUI, LiteLLM, Hermes, Claude Code, OpenAI SDK-style clients and other OpenAI-compatible tools.

The project works through your usual logged-in DeepSeek account in a separate Chrome profile. The local server accepts API requests and then talks to DeepSeek Web itself through the saved browser session.

> ⚠️ This is an experimental web-chat proxy. DeepSeek may change its internal Web API without warning. For production use cases, the official paid DeepSeek API is more reliable.

ForgetMeAI: https://t.me/forgetmeai

---

## Navigation

- [What you get](#-what-you-get)
- [Features](#-features)
- [Quick start](#-quick-start)
- [Windows setup](#-windows-setup)
- [Linux / Chromium setup](#-linux--chromium-setup)
- [VPS / headless setup](#-vps--headless-setup)
- [Permanent service on VPS (systemd)](#-permanent-service-on-vps-systemd)
- [Rootless Podman](#-rootless-podman)
- [Diagnostics / doctor](#-diagnostics--doctor)
- [Session reuse and chat reset](#-session-reuse-and-chat-reset)
- [Multi-account pool](#-multi-account-pool)
- [Ideas for console auth](#-ideas-for-console-auth)
- [Verifying it works](#-verifying-it-works)
- [Request examples](#-request-examples)
  - [Chat Completions](#chat-completions)
  - [Reasoning](#reasoning)
  - [Web search](#web-search)
  - [Streaming](#streaming)
  - [Anthropic Messages API](#anthropic-messages-api)
  - [OpenAI Responses API](#openai-responses-api)
  - [Tool calling](#tool-calling)
- [Models](#-models)
- [Endpoints](#-endpoints)
- [Open WebUI](#-open-webui)
- [Refresh login](#-refresh-login)
- [Project status](#-project-status)

---

## ✨ What you get

- Use DeepSeek Web as a local API endpoint.
- Connect DeepSeek to Open WebUI and other OpenAI-compatible clients.
- Get plain JSON responses or streaming SSE.
- Use reasoning models with separate `reasoning_content`.
- Work with the Anthropic Messages API shim for Claude Code / Anthropic SDK.
- Use the OpenAI Responses API shim for new OpenAI/Codex-style clients.
- Keep separate web sessions for different agents/users.

## 🚀 Features

- **OpenAI-compatible API:** `POST /v1/chat/completions`
- **Anthropic-compatible shim:** `POST /v1/messages`
- **OpenAI Responses shim:** `POST /v1/responses`
- **Streaming:** SSE chunks and plain non-stream JSON responses
- **Reasoning output:** separate `reasoning_content` for thinking models
- **Tool calling:** parses OpenAI tools, Anthropic tools and Responses function tools
- **Model capabilities:** `GET /v1/model-capabilities` with alias → real web mode
- **Agent sessions:** a separate DeepSeek session per `user` / agent id
- **Session recovery:** auto-reset of stale chains/sessions
- **Zero dependencies:** Node.js 18+ for the proxy (`npm run auth` needs Node.js 22+), no npm dependencies

---

## ⚡ Quick start

```bash
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
npm run auth
npm start
```

`npm run auth` opens the auth menu:

1. pick option `1`;
2. log in to DeepSeek in a separate Chrome profile;
3. send a short message such as `ok`;
4. go back to the terminal and press Enter.

> `npm run auth` uses the WebSocket built into Node.js, so it needs
> Node.js 22+. The proxy itself works on Node.js 18+; on older Node, import a
> ready-made `deepseek-auth.json` via `npm run auth:import`.

`npm start` shows the startup menu:

- `1` — log in / refresh DeepSeek login
- `2` — import auth file / cookies
- `3` — show models and statuses
- `4` — start the proxy (default, Enter)
- `5` — exit

Settings can go into `.env` (template: `.env.example`): `npm start`,
`npm run auth` and `npm run doctor` read it themselves. Environment variables
take precedence over values from `.env`.

For a headless/CI start without the menu:

```bash
NON_INTERACTIVE=1 npm start
# or
SKIP_ACCOUNT_MENU=1 npm start
```

By default the server listens on:

```text
http://localhost:9655
```

By default the proxy is reachable only from this computer. For access from the
network, explicitly set an address and a separate proxy key:

```bash
HOST=0.0.0.0 PROXY_API_KEY='replace-with-a-long-random-value' npm start
```

After that, pass the key as `Authorization: Bearer *** or
`x-api-key: *** (that's how the Anthropic SDK sends it). Without
`PROXY_API_KEY`, non-health endpoints remain unauthenticated, so don't expose
such an instance to the network.

Browser requests are allowed from loopback origins. If the UI is open at a
different address, add its exact origin comma-separated, e.g.
`PROXY_CORS_ORIGINS=https://ui.example.com,http://192.168.1.20:3000`.

---

## 🪟 Windows setup

```powershell
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
npm run auth
npm start
```

If Chrome is installed in a non-standard location, set the path explicitly:

```powershell
$env:CHROME_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe"
npm run auth
```

If Chrome is not found, `npm run auth` now prints ready-made instructions for Windows/macOS/Linux instead of a cryptic stack trace.

---

## 🐧 Linux / Chromium setup

```bash
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
CHROME_PATH=$(which chromium) npm run auth
npm start
```

If Chromium is named differently:

```bash
CHROME_PATH=$(which chromium-browser) npm run auth
# or
CHROME_PATH=$(which google-chrome) npm run auth
```

---

## 🖥 VPS / headless setup

The most reliable flow without Chrome on the server:

1. On the home PC that has a GUI/Chrome:

```bash
npm run auth
```

2. Copy `deepseek-auth.json` to the VPS:

```bash
scp deepseek-auth.json user@your-vps:/opt/FreeDeepseekAPI/deepseek-auth.json
```

3. On the VPS, import/verify the file and set safe permissions:

```bash
cd /opt/FreeDeepseekAPI
npm run auth:import -- --input ./deepseek-auth.json
npm run doctor -- --offline
```

4. Start the proxy without the interactive menu:

```bash
NON_INTERACTIVE=1 npm start
```

You can import not only a ready-made `deepseek-auth.json`, but also a browser cookie export:

```bash
DEEPSEEK_TOKEN="<token>" npm run auth:import -- --input ./cookies.json
```

> Important: `deepseek-auth.json` is access to your DeepSeek Web login. Don't commit it, don't publish it, keep it with `0600` permissions.

---

## 🛠 Permanent service on VPS (systemd)

A no-container option: the proxy runs as a system service under a dedicated
user, restarts on failures and starts together with the server. Requires
Node.js 18+ (`node` in `/usr/bin` or `/usr/local/bin`).

```bash
# 1. User and code
sudo useradd --system --home /opt/FreeDeepseekAPI --shell /usr/sbin/nologin freedeepseek
sudo git clone https://github.com/ForgetMeAI/FreeDeepseekAPI /opt/FreeDeepseekAPI

# 2. Secrets: the auth file from `npm run auth` on the home PC and a key for clients
sudo install -d -m 0750 -o root -g freedeepseek /etc/free-deepseek-api
sudo install -m 0600 -o freedeepseek -g freedeepseek ./deepseek-auth.json /etc/free-deepseek-api/deepseek-auth.json
openssl rand -hex 32 | sudo install -m 0600 -o freedeepseek -g freedeepseek /dev/stdin /etc/free-deepseek-api/proxy-api-key

# 3. Service
sudo cp /opt/FreeDeepseekAPI/deploy/free-deepseek-api.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now free-deepseek-api

# 4. Verify
curl -fsS http://127.0.0.1:9655/readyz
sudo -u freedeepseek env DEEPSEEK_AUTH_PATH=/etc/free-deepseek-api/deepseek-auth.json node /opt/FreeDeepseekAPI/scripts/doctor.js
journalctl -u free-deepseek-api -f
```

- The service listens only on `127.0.0.1:9655` and requires the key
  (`Authorization: Bearer *** or `x-api-key`). The key lives in
  `/etc/free-deepseek-api/proxy-api-key`.
- Access from outside — through a reverse proxy with HTTPS (nginx/Caddy) or an
  SSH tunnel. If you need to listen on the network directly, set `HOST=0.0.0.0`
  in `/etc/free-deepseek-api/env` (put any `DEEPSEEK_*` settings there too).
- Update: `cd /opt/FreeDeepseekAPI && sudo git pull && sudo systemctl restart free-deepseek-api`.
- New DeepSeek login: drop the file in with the same command to keep owner and
  permissions, then restart the service:
  `sudo install -m 0600 -o freedeepseek -g freedeepseek ./deepseek-auth.json /etc/free-deepseek-api/deepseek-auth.json && sudo systemctl restart free-deepseek-api`.

---

## 🦭 Rootless Podman

The container is meant only for non-interactive proxy startup. Do the browser
auth on the host with `npm run auth`: the auth scripts and
`deepseek-auth.json` are not copied into the image.

Run Podman as a regular user, without `sudo`.

1. Build the local image:

```bash
podman build --format docker --tag localhost/free-deepseek-api:local --file Containerfile .
```

2. Pass the DeepSeek auth and a separate proxy key through Podman secrets:

```bash
podman secret create --replace free-deepseek-auth ./deepseek-auth.json

printf 'Proxy API key: '
IFS= read -r -s PROXY_API_KEY
printf '\n'
printf '%s' "$PROXY_API_KEY" |
  podman secret create --replace free-deepseek-proxy-key -
```

Use a long random key. The value stays in the `PROXY_API_KEY` variable of the
current shell so you can test the API; it doesn't end up in the image or the
Podman command line.

3. Run the container with minimal privileges:

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

Inside the container, `NON_INTERACTIVE=1`, `HOST=0.0.0.0` and the paths to both
secrets are preset. `REQUIRE_PROXY_API_KEY=1` won't let the container start if
the key secret is missing or empty. On the host the port is published only on
`127.0.0.1`; don't remove that address without a separate network
firewall/access policy.

4. Check liveness, account readiness and a protected endpoint:

```bash
podman healthcheck run free-deepseek-api
curl --fail http://127.0.0.1:9655/readyz
curl --fail \
  -H "Authorization: Bearer ***" \
  http://127.0.0.1:9655/v1/models
```

The built-in healthcheck probes the local `/health` (is the process alive).
`--format docker` is required at build time: in the OCI format (Podman's
default) the `HEALTHCHECK` instruction is silently dropped.
`/readyz` additionally returns `503` if no DeepSeek auth account is currently
ready to serve requests. Container diagnostics:

```bash
podman logs free-deepseek-api
podman inspect --format '{{.State.Health.Status}}' free-deepseek-api
```

Stop and remove the container together with the saved Podman secrets:

```bash
podman stop free-deepseek-api
podman rm free-deepseek-api
podman secret rm free-deepseek-auth free-deepseek-proxy-key
unset PROXY_API_KEY
```

When rotating the auth or the proxy key, replace the corresponding secret and
recreate the container, so the behavior doesn't depend on the Podman version.

---

## 🩺 Diagnostics / doctor

```bash
npm run doctor
# without network requests to DeepSeek:
npm run doctor -- --offline
```

`doctor` checks:

- whether `deepseek-auth.json` / `DEEPSEEK_AUTH_DIR` is found;
- whether the JSON is valid;
- whether `token`, `cookie`, `wasmUrl` are present;
- whether the file permissions are safe on macOS/Linux (`0600`);
- on a normal run — whether the DeepSeek PoW endpoint is reachable.

If you see `data.biz_data is null`, `fetch failed`, `401/403/429`, or Hermes/OpenCode doesn't see the models — run `npm run doctor` first.

---

## ♻️ Session reuse and chat reset

FreeDeepseekAPI doesn't create a new DeepSeek chat for every HTTP request without reason. The logic:

- one `x-agent-session`, `session` or `user` → one DeepSeek chat session;
- if the session id already exists — the proxy reuses it and continues the chain through `parent_message_id`;
- agents (Hermes, Cline, Claude Code, OpenCode…) send the whole history with every request. The proxy remembers which messages are already in the DeepSeek chat, and sends **only the new ones** to the live chat (tool results, the new question) plus a short reminder of the tool-call format. Previously the whole dialogue (up to 80,000 chars) was re-sent on every step, the chat context grew quadratically, and after a dozen steps DeepSeek started answering empty, saying "Content too long", or stopped calling tools (#23, #30);
- if the client changed the history (compacted/edited), changed the system prompt or the tool set — a new DeepSeek chat is opened with the full prompt;
- auto-reset happens on TTL, a DeepSeek session error, a too-long message chain, or when the chat accumulated more than `DEEPSEEK_MAX_SESSION_CHARS` characters (default 3 × `DEEPSEEK_MAX_PROMPT_CHARS`);
- parallel requests from one agent (e.g. background reviews in Hermes) don't mix in one chat: the second request is served in a separate temporary DeepSeek chat;
- the local history is kept as a short context, so a new DeepSeek session can continue the conversation;
- long agent requests are limited to `DEEPSEEK_MAX_PROMPT_CHARS` before sending (default 80,000 chars): the start of the task, fresh tool results and the tool adapter are kept;
- if the client already sent a multi-turn history, the local recovery history isn't added a second time;
- an empty response is retried at most `DEEPSEEK_MAX_RETRIES` times (default 2), with the context shrunk on each retry;
- a long generation (thinking) isn't cut off by timeout: the stream is bounded by a pause without data (`DEEPSEEK_STREAM_IDLE_TIMEOUT_MS`, default 60 s) and the overall limit `DEEPSEEK_STREAM_MAX_MS` (10 minutes).

Set the agent/session explicitly:

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-agent-session: my-agent" \
  -d '{"model":"deepseek-chat","messages":[{"role":"user","content":"Hi"}]}'
```

List the active sessions:

```bash
curl http://localhost:9655/v1/sessions
```

Reset one session:

```bash
curl -X POST "http://localhost:9655/reset-session?agent=my-agent"
```

Reset all sessions:

```bash
curl -X POST "http://localhost:9655/reset-session?agent=all"
```

Why chats still appear in DeepSeek Web: the proxy works through the internal Web Chat API, and DeepSeek stores the real chat sessions on its side. That's normal for a web proxy. The point of session reuse is not to spawn new chats without need and to reset cleanly only when a chain went stale/broke.

---

## 👥 Multi-account pool

You can connect multiple auth files. The right model: sticky account per agent/session — the proxy doesn't switch accounts inside a live DeepSeek session. If an account got `401/403/429` and went into cooldown, the session is safely reset and a new request can move to another available account.

Option 1 — a directory with auth files:

```bash
mkdir -p accounts
cp deepseek-auth-main.json accounts/main.json
cp deepseek-auth-backup.json accounts/backup.json
chmod 600 accounts/*.json
DEEPSEEK_AUTH_DIR=./accounts NON_INTERACTIVE=1 npm start
```

Option 2 — a list of files:

```bash
DEEPSEEK_AUTH_PATH="./accounts/main.json,./accounts/backup.json" NON_INTERACTIVE=1 npm start
```

How the pool works:

- a new agent/session gets an available account round-robin;
- the chosen account is pinned to the session (`sticky`);
- on `401`, `403`, `429` the account goes into cooldown and the request is retried on another ready account;
- DeepSeek's "Too many messages" reply also counts as a rate limit: the account goes into a short cooldown (`DEEPSEEK_RATE_LIMIT_COOLDOWN_MS`, default 60 s); if there are no other accounts, the client gets `429` with `Retry-After`;
- if a session's sticky account went into cooldown, the old DeepSeek session is reset, so a rate-limited/expired account isn't hammered;
- account status is visible in `/health` without auth-file paths and without file names;
- auth files must be kept with `0600` permissions.

Configure the cooldown:

```bash
DEEPSEEK_ACCOUNT_COOLDOWN_MS=600000 npm start
```

---

## 🔑 Ideas for console auth

The password flow from PR #3 is doable, but it's safer not to store the password and not to make it the default. A sane implementation:

1. `npm run auth:console` asks for email/phone and password through a hidden prompt.
2. The password lives only in process memory, never written to files/logs/history.
3. The script replays the Web login flow through `fetch`/CDP: it gets a captcha/verify challenge, gives the human a link/code, waits for confirmation.
4. After a successful login, only the standard-format `deepseek-auth.json` is saved.
5. If DeepSeek asks for captcha/2FA — the script honestly says "open the link, pass the check, press Enter" instead of trying to bypass the protection.
6. For a VPS, the `auth:console --no-save-password --output deepseek-auth.json` mode is better.

Minimal safe MVP: console auth is interactive-only, no env password. An acceptable automation variant: `DEEPSEEK_EMAIL=... npm run auth:console`, but the password is still entered through a hidden prompt.

---

## ✅ Verifying it works

```bash
curl http://localhost:9655/
curl http://localhost:9655/v1/models
curl http://localhost:9655/v1/model-capabilities
```

If everything is fine, `/health` returns the server status, the list of supported aliases and `config_ready: true`.

---

## 🧪 Request examples

### Chat Completions

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [{"role": "user", "content": "Hi! Answer in one sentence."}],
    "stream": false
  }'
```

### Reasoning

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-reasoner",
    "messages": [{"role": "user", "content": "Answer briefly: why is the sky blue?"}],
    "stream": false
  }'
```

With `stream: true`, a normal answer (a request without `tools`) is streamed to
the client as DeepSeek writes it: first the thinking, then the text. If the
request has `tools`, the answer may turn out to be a tool call, so the proxy
waits for the end and returns it whole — and while waiting, every
`DEEPSEEK_STREAM_KEEPALIVE_MS` (10 s) it sends the SSE comment
`: keep-alive`, so the client doesn't drop the connection on timeout. An error
that happened after the stream started arrives as an `error` event inside the
stream (the OpenAI and Anthropic SDKs turn it into an exception); before the
stream starts — as a normal HTTP status.

For reasoning models the API returns the thinking chain separately from the final answer:

- non-stream: `choices[0].message.reasoning_content`
- stream: `choices[0].delta.reasoning_content`
- usage: `usage.completion_tokens_details.reasoning_tokens`

`reasoning_tokens` is an approximate estimate from the extracted DeepSeek Web `THINK` text, because the web stream doesn't return the official token usage for reasoning separately.

### Web search

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat-search",
    "messages": [{"role": "user", "content": "Find a fresh fact about DeepSeek and answer briefly."}],
    "stream": false
  }'
```

### Streaming

```bash
curl -N -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [{"role": "user", "content": "Write a short joke."}],
    "stream": true
  }'
```

### Anthropic Messages API

```bash
curl -X POST http://localhost:9655/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "max_tokens": 512,
    "messages": [{"role": "user", "content": "Answer exactly OK"}],
    "stream": false
  }'
```

For Claude Code you can point the backend directly:

```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:9655"
export ANTHROPIC_AUTH_TOKEN="dummy-key"
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
claude --model deepseek-chat
```

### OpenAI Responses API

```bash
curl -X POST http://localhost:9655/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "input": "Answer exactly OK",
    "stream": false
  }'
```

### Tool calling

FreeDeepseekAPI accepts:

- OpenAI `tools`;
- Anthropic `tools`;
- Responses API function tools.

The proxy asks DeepSeek to return a strict JSON tool call, but can also parse fallback formats:

- `TOOL_CALL:`
- fenced JSON with an explicit `tool_call`, `tool_calls`, or `function_call` envelope
- `<tool_call>...</tool_call>`
- DeepSeek DSML (`<｜DSML｜tool_calls>...`) and the Web variant with `<｜｜DSML｜｜ Tool Calls>`

Typical model mistakes are fixed during parsing: unescaped backslashes in
Windows paths (`"C:\git\src\index.ts"`) and real line breaks inside string
arguments. Code blocks in another language (```` ```csharp ````,
```` ```python ````) are never treated as a tool call.

---

## 🧠 Models

`GET /v1/models` returns only the aliases that are currently verified and working through this proxy.

### Unified DeepSeek Web mode (since 2026-09-10)

On September 10, 2026 DeepSeek merged the web-chat modes "Fast", "Expert" and
"Recognition" into one smart mode on the **DeepSeek-V4.1-Flash** model, and on
September 14 disabled V4 Pro. The model switcher is gone; only
"Deep thinking" (`thinking_enabled`) and "Search" (`search_enabled`) remain (#31).

So all aliases now send the same `model_type: "default"` to the Web API and
differ only by those two flags. The old names (`deepseek-expert`,
`deepseek-v4-pro`, …) are kept so client configs don't break, and are marked as
`deprecated` in `/v1/model-capabilities`. If DeepSeek changes the `model_type`
value again, it can be overridden through `DEEPSEEK_MODEL_TYPE` (an empty value
means don't send the field).

### Working aliases

| Alias | Deep thinking | Web search | Comment |
| --- | --- | --- | --- |
| `deepseek-chat` | no | no | basic chat |
| `deepseek-default` | no | no | compatible alias |
| `deepseek-v4-flash` | no | no | official API name |
| `deepseek-v4.1-flash` | no | no | current web-chat model |
| `deepseek-v3` | no | no | legacy alias |
| `deepseek-reasoner` | yes | no | `thinking_enabled=true` |
| `deepseek-r1` | yes | no | R1-compatible alias |
| `deepseek-chat-search` | no | yes | web search |
| `deepseek-default-search` | no | yes | web search alias |
| `deepseek-reasoner-search` | yes | yes | reasoning + search |
| `deepseek-r1-search` | yes | yes | R1-compatible + search |
| `deepseek-expert` | yes | no | legacy: "Expert" merged into the unified mode |
| `deepseek-v4-pro` | yes | no | legacy: V4 Pro disabled, V4.1-Flash answers |
| `deepseek-expert-search` | yes | yes | legacy alias, now works |

### Images (verified against the live DeepSeek Web API)

On October 7, 2026, PNG/JPEG upload through `deepseek-vision` was verified on
the live DeepSeek Web API: multipart upload, file processing from `PENDING`
through `PARSING` to `SUCCESS`, the file ID referenced in the completion, and a
visual answer with readable text, colors and element placement. The model
called almost-square rectangles squares, so shape-recognition accuracy should
be checked for the specific task. Image upload is enabled by default. To forbid
sending images to DeepSeek, explicitly set `DEEPSEEK_IMAGE_UPLOAD=0`:

```bash
DEEPSEEK_IMAGE_UPLOAD=0 npm start
```

- inline images are accepted: OpenAI `image_url` / Responses `input_image` with
  `data:image/...;base64,...` and Anthropic `image` with
  `source.type: "base64"` (including inside `tool_result`);
- `https://...` links are not downloaded (SSRF protection) — the model only
  sees the address;
- PNG, JPEG, WebP and GIF are supported; other formats (e.g. SVG) stay as a
  text marker;
- only the images of new messages are uploaded into an already-open chat; into
  a new chat — the last `DEEPSEEK_MAX_IMAGES` (4) images of the conversation;
  one image — up to `DEEPSEEK_MAX_IMAGE_BYTES` (7 MB), the whole request — up
  to 10 MB; already-uploaded images aren't uploaded again (30-minute cache);
- if DeepSeek couldn't process an image, the request fails with `502
  image_upload_failed` instead of a silent "blind" answer;
- with upload enabled, `deepseek-vision` appears in `/v1/models` (it's the same
  unified mode).

With `DEEPSEEK_IMAGE_UPLOAD=0`, images are replaced with a short marker
`[Image attached; not visible through this proxy]`; their bytes aren't uploaded
to DeepSeek.

The full mapping:

```bash
curl http://localhost:9655/v1/model-capabilities
```

---

## 🔌 Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/` or `/health` | proxy status |
| `GET` | `/v1/models` | list of working OpenAI-compatible aliases |
| `GET` | `/v1/model-capabilities` | full mapping of aliases, real model, capabilities |
| `POST` | `/v1/chat/completions` | OpenAI-compatible Chat Completions |
| `POST` | `/v1/messages` | Anthropic Messages API shim |
| `POST` | `/v1/responses` | OpenAI Responses API shim |
| `GET` | `/v1/sessions` | active local agent sessions |
| `POST` | `/reset-session?agent=<id>` | reset one session |
| `POST` | `/reset-session?agent=all` | reset all sessions |

---

## 🖥 Open WebUI

Base URL for Open WebUI in Docker:

```text
http://host.docker.internal:9655/v1
```

For a local run without Docker:

```text
http://localhost:9655/v1
```

If `PROXY_API_KEY` is not set, the API key can be anything. If the key is set,
the client must pass exactly that key — the proxy checks the bearer token
before granting access to models, sessions and completions.

---

## 🔐 Refresh login

```bash
npm run auth
npm start
```

If DeepSeek started answering `401`, `403` or asks for a new PoW/session — run `npm run auth` again and refresh the saved browser session.

Local auth files must not end up on GitHub:

- `deepseek-auth.json`
- `.chrome-for-testing-profile-deepseek/`
- `.env`

They're already in `.gitignore`.

---

## 🧪 Tests

Syntax check, unit tests and end-to-end tests against a local mock DeepSeek
server (no network and no account needed; `node --test` needs Node.js 18.13+):

```bash
npm test
```

Live smoke tests against a running local proxy (if `PROXY_API_KEY` is set, it's
taken from the environment):

```bash
BASE_URL=http://127.0.0.1:9655 MODEL=deepseek-chat npm run test:live
```

---

## 📌 Project status

FreeDeepseekAPI is an experimental web-chat proxy for local use and integrations. It depends on the current DeepSeek Web Chat contract, so when DeepSeek changes things on its side, the auth/session logic or the model mapping may need an update.

If something stopped working:

1. refresh the login with `npm run auth`;
2. check `/v1/model-capabilities`;
3. retry the request on a fresh session;
4. if the problem persists — DeepSeek most likely changed its internal Web API.

---

<p align="center">
  <strong>ForgetMeAI</strong> · <a href="https://t.me/forgetmeai">Telegram</a>
</p>
