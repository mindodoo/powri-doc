# Story UX-12: Auto app version from last done story

Status: ready-for-dev

Version when done: **2.12.2.12**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As the product owner,  
I want the displayed app version to follow the last shipped story,  
so that I stop hand-editing `NEXT_PUBLIC_APP_VERSION` / `appMeta.ts` every time.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-12, §5, D5).

## Acceptance Criteria

1. Display format for this epic: **`2.12.2.{n}`** where `{n}` is the UX story number (this story → `2.12.2.12` when **done on `main`**).
2. **Source of truth:** last `development_status` entry in `_powri/implementation-artifacts/sprint-status.yaml` that is a **story** (not `epic-*`, not retrospective) with status **`done`**. **Ignore** `in-progress` / `ready-for-dev` / `review`.
3. Mapping: `ux-{n}-*` → `2.12.2.{n}`. Numbered keys `12-4-*` stay three-part `2.12.4` if that is the latest done **before** any `ux-*` is done. Once an `ux-*` is done, prefer the highest `ux-n` done (do not compare `2.12.2.1` vs `2.12.4` with naive semver).
4. **Build-time** inject into `getAppVersion()` default. `NEXT_PUBLIC_APP_VERSION` env **overrides** for emergencies (CI already sets `ci` in Playwright).
5. **Stop** requiring humans to edit `DEFAULT_APP_VERSION` in `appMeta.ts` each story.
6. About page (`info/page.tsx`) and analytics `app_version` use the same helper (analytics may keep env fallback — align so they don’t drift).

## Testing & Definition of Done

- [ ] **Unit (Vitest):** Parser: given a fixture YAML snippet, `ux-3-…: done` + `ux-4-…: ready-for-dev` → `2.12.2.3`; `12-4-…: done` and no ux done → `2.12.4`
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** `commonProperties` `app_version` — update if it bypasses `getAppVersion()` (`process.env.NEXT_PUBLIC_APP_VERSION ?? '1.0.0'` today — **wire through helper** so About and PostHog match)
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** `npm run build`; About shows derived version when env unset
- [ ] **Manual QA:** N/A beyond About string

## Tasks / Subtasks

- [ ] Version parser module under `web/src/lib/site/` + tests (AC: 2–3)
- [ ] Build script or `getAppVersion()` reading generated `web/src/lib/site/generatedAppVersion.ts` (do not read YAML from the Edge client at request time if the file isn’t in the Vercel root) (AC: 4)
- [ ] `prebuild` / `npm run build` generates the constant from `../_powri/implementation-artifacts/sprint-status.yaml` (Vercel Root Directory `web` — path must work: `../_powri/...` from `web/`) (AC: 4)
- [ ] Align `commonProperties.ts` (AC: 6)
- [ ] Document override in `web/.env.example`

## Dev Notes

**Today:** `appMeta.ts` `DEFAULT_APP_VERSION = '2.12.4'`; `.env.local` may set `NEXT_PUBLIC_APP_VERSION`; Playwright `ci`; `commonProperties` duplicates a different fallback (`1.0.0`).

**Vercel:** Root Directory is `web`. Generator must run in `web/package.json` `prebuild`/`predev` and resolve YAML at `../_powri/implementation-artifacts/sprint-status.yaml`. Commit generated file **or** generate in CI — prefer generate on build so `main` always matches sprint-status; commit the generator, gitignore output **only if** local/CI always run prebuild (Playwright needs a version — `ci` override is enough).

**Do not:** Bump from the story currently in progress. Semver-compare `2.12.2.x` against `2.12.4`.

### Files

| Path | Role |
|------|------|
| `web/src/lib/site/appMeta.ts` | Consume generated + env |
| `web/src/lib/analytics/commonProperties.ts` | Use `getAppVersion()` |
| `web/src/app/[locale]/info/page.tsx` | Already uses helper |
| `web/scripts/` or `web/src/lib/site/appVersionFromSprint.ts` | NEW |
| `web/package.json` | prebuild |
| `web/.env.example` | Override note |

### References

- PRD §5, D5
- `web/playwright.config.ts` `NEXT_PUBLIC_APP_VERSION: 'ci'`

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
