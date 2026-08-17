# AGENTS.md

## Cursor Cloud specific instructions

`vibe-blog` is a multi-Agent AI blog generator with two services that run together in development:

- Backend: Flask app in `backend/` (dev server on port `5001`). Entry point `backend/app.py`.
- Frontend: Vue 3 + Vite in `frontend/` (dev server on port `5173`). Vite proxies `/api` and `/outputs` to `http://localhost:5001` (see `frontend/vite.config.ts`), so both services must run for the UI to work.

Python deps are installed into a virtualenv at `backend/venv` (Python 3.12), and the update script keeps it and the frontend `node_modules` in sync. `python3.12-venv` is a required system package (installed during environment setup, captured in the snapshot).

### Running the services (non-obvious)

- Backend must be started with `PYTHONPATH` pointing at the `backend/` directory, because the code imports as `from services ...` / `from routes ...` (backend dir is the import root). From `backend/`: `PYTHONPATH=/workspace/backend venv/bin/python app.py`.
- Frontend: from `frontend/`, `npm run dev` (Vite on 5173). Backend health check: `curl http://localhost:5001/health`.

### Testing / build (standard commands)

- Backend tests: from `backend/`, `PYTHONPATH=/workspace/backend venv/bin/pytest -m "not llm"`. LLM-marked tests are skipped by default (they need real API keys); this matches CI (`.github/workflows/test-backend.yml`). `pytest.ini` auto-adds coverage reporting.
- Frontend tests: from `frontend/`, `npm test -- --run` (Vitest). Build: `npm run build`. See `TESTING.md`.

### API keys / degraded mode (important)

- The backend starts and serves APIs with NO keys configured, but logs warnings and runs in degraded mode (search, image, PDF, video features disabled).
- Core blog generation requires an OpenAI-compatible LLM key. Without it, a generation request fails immediately in the UI with `博客生成请求已不可用` ("blog generation request unavailable"). Set `OPENAI_API_KEY` (and optionally `OPENAI_API_BASE` / `TEXT_MODEL`) for end-to-end generation. Optional integrations: `ZAI_SEARCH_API_KEY` (web search), `NANO_BANANA_API_KEY` (AI images), `MINERU_TOKEN` (PDF parsing). Copy `backend/.env.example` to `backend/.env` to configure; a `.env` is not required just to start the services.
- `diskcache` is not installed (not in `requirements.txt`); the researcher cache is disabled with a harmless warning.
