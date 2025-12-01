# Charity Recommender

Lightweight, production-ready scaffold for a static GitHub Pages frontend (`docs/`) and a FastAPI backend (`api/`) that delivers rotating daily picks plus survey-driven recommendations without storing PII.

## Repository layout

- `docs/` – Static site ready for GitHub Pages (`docs/` folder). Includes `index.html`, `survey.html`, `privacy.html`, shared assets, and a `config.json` that points the frontend at the deployed API.
- `api/` – FastAPI service with `/api/recommend`, `/api/daily-picks`, and `/api/status`. Stateless cursors are HMAC-signed and expire after 10 minutes. A small curated pool stands in for Every.org until the live integration is ready.
- `Test.ipynb` – Scratchpad currently unused by the app.

## Frontend quick start

1. Update `docs/config.json` with your deployed API base URL (for local dev keep the default `http://localhost:8000`).
2. Serve the `docs/` directory with any static server (for preview: `python -m http.server --directory docs 4173`).
3. Open `http://localhost:4173/index.html` and run through the survey. Errors surface in accessible alert regions and do not clear the form.

Deploy to GitHub Pages by enabling Pages (Source: `main`, folder: `/docs`). The survey is keyboard accessible, uses ARIA live regions for errors, and includes focus management after results render.

## Backend quick start

```bash
cd api
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Environment variables:

- `API_BASE` – Public URL for the API (used in `/api/status` metadata).
- `EVERYORG_API_KEY` – Reserved for the future Every.org integration (optional today).
- `CORS_ALLOW_ORIGIN` – Comma-separated list of allowed origins (set to your Pages domain in production).
- `APP_ENV` – `development`, `preview`, or `production`.
- `SECRET_KEY` – Required for signing cursors (defaults to a dev-safe string; override in prod).
- `RATE_LIMIT_PER_MINUTE`, `CURSOR_TTL_SECONDS` – Tunable rate limiting and cursor expiration.

### API surface

- `GET /api/status` → `{ "ok": true, "version": "<git-sha>", "env": "development" }`
- `GET /api/daily-picks?limit=3` → Deterministic daily rotation seeded by UTC date + secret key.
- `POST /api/recommend` → Accepts survey answers + optional cursor; returns up to `limit` charities plus a signed cursor. Cursors embed `{page, page_size, signature, issued_at}` and become invalid after 10 minutes.

Rate limiting is enforced at 60 req/min/IP by default. Responses degrade gracefully (cursor expiry returns `cursor: null` with rationale) and all CORS settings are locked to your configured domain.

## Data sources & next steps

- Current recommendations use the curated pool in `api/app/data.py`. Swap in Every.org’s search endpoint once API keys are available.
- Update styling/content in `docs/assets/styles.css` as needed while preserving the documented DOM contract.

Reference: https://docs.every.org/docs/endpoints/nonprofit-search
