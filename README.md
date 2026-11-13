# BDN Election Results: 2026

Live election results app for the Bangor Daily News covering Maine's June 9, 2026 primary, with race pages, town-by-town results, maps and a live-updates feed. Results data comes from [Decision Desk HQ](https://decisiondeskhq.com).

> **Fork:** this repo is a fork of [mikeshepherdme/bdn-election-results](https://github.com/mikeshepherdme/bdn-election-results). Development happens upstream; this fork has no commits of its own beyond this README.

See [`DEVELOPER.md`](DEVELOPER.md) for the full developer handoff (data flow, slug format, live-update system, go-live checklist). Note that parts of it predate later changes (for example, the town list is now served as the static file `public/towns.json` rather than an `/api/towns` route).

## Stack

Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS v4, Leaflet/react-leaflet and d3-geo for maps, SWR for client polling, Python 3 for the data pipeline.

## How it works

- **Race data:** `lib/mock-data.ts` imports `data/ddhq_races.json` (DDHQ-shaped race objects; the checked-in file is flagged `test_data`) and normalizes it into the types in `lib/types.ts`.
- **Pages:** `app/page.tsx` (homepage, races grouped by level), `app/races/[slug]` (race detail with table, map and municipality breakdown), `app/towns/[slug]` (all races for one town).
- **API routes:** `/api/race/[slug]` (race data), `/api/race/[slug]/events` (GET events; POST/DELETE manual notes and story links, bearer-token protected), `/api/race/[slug]/auto-updates` (generates call, milestone, big-vote-drop and bellwether events). Events are stored in `data/race-events.json`.
- **DDHQ poller:** `ddhq_poller.py` authenticates to the DDHQ API with OAuth2 client credentials, pulls Maine races, race details and race calls, and upserts them into Supabase (`races`, `candidates`, `vcu_results`, `race_snapshots`; schema in `data/schema.sql`). Flags: `--dry-run`, `--loop` (every 60 seconds).
- **Geo and historical data:** town boundaries in `data/geo/` and `public/geo/` (including per-district House and Senate GeoJSON); 2018 gubernatorial primary results and voter registration by town in `data/historical/`, built by `process_historical.py` and `process_geojson.py` from local source files.
- **Prototype:** `prototype.html` and `generate_pages.py` are an earlier static-HTML prototype; generated `races/` and `towns/` output is git-ignored.

## Run locally

```bash
npm install
npm run dev        # http://localhost:3000
npm run build      # production build
```

Poller (requires `requests`):

```bash
python3 ddhq_poller.py --dry-run
```

## Environment variables

Set in `.env.local` (git-ignored). Names only:

- `DDHQ_CLIENT_ID`, `DDHQ_CLIENT_SECRET`, `DDHQ_ELECTION_ID`: DDHQ API access (poller)
- `DDHQ_API_BASE`, `DDHQ_RACE_DATE`: listed in `DEVELOPER.md`
- `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`: Supabase writes (poller)
- `UPDATE_PASSWORD`: bearer token for posting/deleting manual live-update events

## Deployment

`.github/workflows/deploy.yml` builds on push to `main` and uploads `out/` to GitHub Pages. GitHub Pages is not enabled on this fork, so there is no live URL here. Upstream commit history references Vercel deployment.
