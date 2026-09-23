# Base44 Dev Environment

## What this is
白板声画工坊 — a local AI video production workbench. Fullstack: Python FastAPI backend + Vite/vinext (React RSC) frontend.

## Architecture
- **Backend**: `webapp/server.py` — FastAPI on port 18765. Serves `/api/*`. No boot-time env vars required; API keys for external services (OpenAI-compatible text/image models, IndexTTS Gradio endpoint) are configured at runtime via the UI and stored in `.webapp/config.json`.
- **Frontend**: `web/` — Vite + vinext (Next.js-style RSC) dev server on port 13000. Proxies `/api/*` to the backend via `CS_BOARD_API_URL` env var (defaults to `http://127.0.0.1:18765`).
- The frontend uses `@cloudflare/vite-plugin` with D1/R2 bindings set to `null` in `web/.openai/hosting.json` — no database needed for dev.

## Running
```
docker compose -f docker-compose.base44.yml up -d --build
```
- Frontend: host port 3000 → container 13000
- Backend: internal only (port 18765, not exposed publicly)
- Health check: `curl http://127.0.0.1:18765/api/health` (backend), `curl http://127.0.0.1:13000` (frontend)

## Secrets
No external secrets required to boot. All API credentials are entered through the app's settings UI at runtime.

## Limitations in this dev environment
- The video rendering pipeline (`scripts/render_stream_whiteboard.py`, `video_renderer/` Remotion) requires opencv-python, numpy, PyAV, Pillow, and Node.js — not installed in the minimal backend container. The UI loads and settings can be configured, but actual video generation will fail at the rendering step.
- TTS requires a separate IndexTTS Gradio/FastAPI service (default `http://127.0.0.1:7860`) not included in this compose setup.

## Key env vars
| Var | Default | Purpose |
|-----|---------|---------|
| `CS_BOARD_ROOT` | repo root | Where the app finds assets, scripts, pronunciation.yaml |
| `CS_BOARD_STATE_DIR` | `.webapp/` | Jobs, voices, styles, config.json |
| `CS_BOARD_API_URL` | `http://127.0.0.1:18765` | Vite proxy target for `/api/*` |
| `CS_BOARD_NODE` | `node` (from PATH) | Node binary for Remotion rendering |
