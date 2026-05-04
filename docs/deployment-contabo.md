# Contabo Deployment — Readiness Notes

**Date:** 2026-05-04
**Host:** Contabo VPS (IPv6: `2407:3641:2314:3880::1`)
**Hermes Agent:** NousResearch upstream, installed via official installer

## Current State

Both APIs verified healthy and bound to `0.0.0.0`:

| Service | Port | Status | Notes |
|---------|------|--------|-------|
| Gateway API | `8642` | ✅ Healthy | `/health` returns `{"status":"ok","platform":"hermes-agent"}` |
| Dashboard API | `9119` | ✅ Healthy | `/api/status` returns v0.12.0 (2026.4.30) |

## Auth

- Gateway API requires authentication — `/v1/models` returns `401` without valid token
- No `~/.hermes/.env` file exists; config is entirely in `~/.hermes/config.yaml`
- `api_server` section in config is empty (`{}`); gateway read from `gateway.platforms.api_server`
- Gateway log (2026-05-02) shows: "No API key configured. All requests will be accepted without authentication" — but current 401 suggests a key was added since then
- **For workspace:** set `HERMES_API_TOKEN` to the same value as the gateway's `API_SERVER_KEY`

## Networking

- Both ports listen on `0.0.0.0` — reachable from any interface
- No Tailscale installed
- IPv6-only public IP (no IPv4 on this VPS)
- Firewall status not verified — confirm ports 8642/9119/3000 are open before remote access

## Workspace Deployment Paths

### Option A: Deploy workspace on same server (recommended)

```bash
cd ~/hermes-workspace
cp .env.example .env
echo 'HERMES_API_URL=http://127.0.0.1:8642' >> .env
echo 'HERMES_DASHBOARD_URL=http://127.0.0.1:9119' >> .env
# If gateway auth is enabled:
# echo 'HERMES_API_TOKEN=<your-key>' >> .env
pnpm dev  # serves on :3000
```

Then access from browser via reverse proxy, SSH tunnel, or direct port exposure.

### Option B: Remote workspace → this server

Point workspace `.env` at the server's public IP:
```bash
HERMES_API_URL=http://<server-ip>:8642
HERMES_DASHBOARD_URL=http://<server-ip>:9119
```

Requires firewall to allow 8642 and 9119 from client IP.

## Prerequisites Checked

- ✅ Node.js 22+ (verify with `node --version` if deploying)
- ✅ `pnpm` (verify with `which pnpm` if deploying)
- ✅ `gh` CLI authenticated (can clone repos)
- ⚠️ API auth token needed if `API_SERVER_KEY` is set on gateway
