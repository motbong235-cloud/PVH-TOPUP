# PVH TOPUP — deploy package

## Files
| File | Purpose |
|---|---|
| `server_v16.py` | Flask backend — payment (CamRapidPay/KHQR) + auto top-up (Khmer TopUp) |
| `index_v6.html` | Storefront frontend, served at `/` |
| `db_default.json` | Seed data — one demo Free Fire product so the site isn't empty on first boot |
| `requirements.txt` | Python dependencies |
| `Procfile` | Start command for Render/Heroku-style platforms |
| `render.yaml` | One-click Render Blueprint |
| `.env.example` | Every environment variable the server reads |
| `manifest.json` | Minimal PWA manifest |

**Not included** — bring your own if you use them, the server degrades gracefully without them:
- `admin_v5.html` (the `/admin` UI — every `/api/admin-*` endpoint still works without it, e.g. via curl/Postman with the `x-admin-token` header)
- `icon-192.png` / `icon-512.png`

## Deploy on Render
1. Push these files to a GitHub repo.
2. Render dashboard → **New → Blueprint** → point at the repo (uses `render.yaml`).
3. Fill in the secret env vars Render will prompt for: `KHMERTOPUP_API_KEY`, `CAMRAPID_API_KEY`, `TELEGRAM_BOT_TOKEN`, `ADMIN_CHAT_ID`. `ADMIN_PANEL_TOKEN` and `FRONTEND_PAYLOAD_KEY` are auto-generated.
4. Deploy. Your site is live at the Render URL; admin API at `/api/admin-*` with header `x-admin-token: <ADMIN_PANEL_TOKEN>`.

No `render.yaml`? Create a plain **Web Service** instead, set the build command to `pip install -r requirements.txt` and the start command to the line in `Procfile`, then add the same env vars by hand.

## Wire up Khmer TopUp
1. Get your key at https://khmer-topup.com/settings, set it as `KHMERTOPUP_API_KEY`, and top up the wallet balance there (orders are charged against it).
2. `GET /api/admin-khmertopup-games` (with `x-admin-token`) to pull the live catalogue.
3. For each game in your admin panel/db: set `khmertopup_slug` (from that catalogue) and `has_server_id` (true if `server_label` isn't null).
4. For each product: set `provider_package` to the matching `package_id` (integer).
Until a product has both set, it just falls back to manual delivery — nothing breaks.

## Run locally
```
pip install -r requirements.txt --break-system-packages
cp .env.example .env   # fill in your keys
python server_v16.py
```
Site: http://localhost:5000/ — Admin API: http://localhost:5000/api/admin-games (header `x-admin-token`)

## Make the whole site private (optional)
Set `SITE_PRIVATE=true`, `SITE_USERNAME`, `SITE_PASSWORD` — the entire site (not just `/admin`) then requires an HTTP login before it loads anything, storefront and API included.
