# Story 6.7b: All Resort Unsplash Images (API batch)

Status: done

baseline_commit: 4f5f45e

## Story

Complete **Phase B** of Story 6.7 — populate hero/card Unsplash images for all remaining resorts via the official API.

## Approach

Use `scripts/fetch_unsplash_assets.py` with `--hero-only --skip-existing --apply`:

- **Search** — Unsplash `/search/photos` per resort editorial query
- **Attribution** — full metadata written to resort markdown
- **Download trigger** — `GET /photos/:id/download` per Unsplash §17.2 compliance
- **CDN map** — `urls.regular` merged into `web/src/lib/content/unsplash-cdn-map.json`

## Prerequisites

1. `UNSPLASH_ACCESS_KEY` in project root `.env` (Access Key, not Application ID)
2. `UNSPLASH_UTM_SOURCE=japan_ski_planner` (default)

## Run

```bash
cp .env.example .env   # add your Access Key
cd web && npm run fetch:unsplash
```

Or dry-run first:

```bash
python3 scripts/fetch_unsplash_assets.py --hero-only --skip-existing
```

## Tasks

- [x] Extend `RESORT_QUERIES` to all 20 slugs
- [x] `--hero-only`, `--skip-existing`, auto CDN map generation
- [x] Shared `unsplashCdn.ts` + JSON map for image URLs
- [x] Run API batch for 17 remaining resorts (needs `.env` key)
- [x] Repair hero rows from card (apply bug fix)
- [x] Re-fetch 7 resorts with fallback queries
- [x] Dev cache fix — reload resort content without server restart
- [x] `test:launch` — zero "no Unsplash images" warnings
