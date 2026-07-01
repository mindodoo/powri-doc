# Story 1.3: Markdown Content Loader & Resort Types

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **developer**,
I want resort markdown imported at build time with typed frontmatter,
So that all features read from a single content source of truth.

## Acceptance Criteria

1. **Given** files in `content/resorts/<slug>.md` following `content-model.md` schema  
   **When** the build runs  
   **Then** a content loader parses YAML frontmatter + markdown body via gray-matter (NFR-5)

2. **Given** parsed resort files  
   **When** `getAllResorts()` (or equivalent) is called  
   **Then** only resorts with `content_status: published` are returned

3. **Given** PRD §8.1 / `content-model.md` §4  
   **When** TypeScript types and Zod schema are inspected  
   **Then** they cover required fields including `images`, `trail_map_url`, `transport_options`, `food_picks`, gateway/curated picks, and season-labelled T2 fields (`season_confirmed`, `lift_pass_adult_day`, etc.)

4. **Given** unpublished resort slugs exist (all 20 currently `draft`)  
   **When** the loader runs in development or build  
   **Then** a warning is logged per unpublished slug and the build does **not** fail

## Tasks / Subtasks

- [x] **Add dependencies** (AC: 1)
  - [x] `npm install gray-matter zod` in `web/`
  - [x] Do not add CMS, database, or runtime markdown fetching

- [x] **Types & schema** (AC: 3)
  - [x] Create `web/src/lib/content/types.ts` — `Resort`, `ResortBody`, `ResortImage`, transport/pick sub-types (camelCase TS)
  - [x] Create `web/src/lib/content/schema.ts` — Zod schema mirroring YAML frontmatter (snake_case); validate at parse boundary
  - [x] Map snake_case frontmatter → camelCase `Resort` in loader (per architecture naming rules)

- [x] **Content loader** (AC: 1, 2, 4)
  - [x] Create `web/src/lib/content/paths.ts` — resolve `content/resorts/` relative to repo root (`../content` from `web/` cwd)
  - [x] Create `web/src/lib/content/parseResortBody.ts` — gray-matter parse + body section split (`Quick verdict`, `Highlights`, etc.)
  - [x] Create `web/src/lib/content/loadResorts.ts` — read all `*.md` except `_TEMPLATE.md`; filter published; dev/build warnings for skipped slugs
  - [x] Create `web/src/lib/content/index.ts` — export `getAllResorts()`, `getResortBySlug(slug)`, `getUnpublishedSlugs()` (dev helper)

- [x] **Build-time verification** (AC: 1, 2, 4)
  - [x] Import loader in `web/src/app/page.tsx` (server component) — display published count; proves build-time execution
  - [x] Run `npm run build` — must pass with 0 published resorts (warnings for 20 drafts expected)

- [x] **Validate** (AC: 1–4)
  - [x] `npm run build` passes
  - [x] `npm run lint` passes
  - [x] Console shows `[content]` warnings for unpublished slugs during build

- [x] **Out of scope — do NOT implement**
  - Resort detail pages / `[slug]` routes (Story 2.3 / 3.x)
  - Publishing resort content (editorial — all 20 remain `draft`)
  - Site images loader (`content/site-images.md`) — later story
  - Client-side markdown fetch
  - Changing content files in `content/resorts/`

## Dev Notes

### Critical constraints

| Rule | Detail |
|------|--------|
| **Content location** | Read from repo-root `content/resorts/` — **never** copy markdown into `web/` |
| **Path resolution** | `path.join(process.cwd(), '..', 'content', 'resorts')` when cwd is `web/` |
| **Published filter** | `content_status === 'published'` only in `getAllResorts()` |
| **Draft handling** | Log warning per skipped slug; **do not fail build** when zero published (current state) |
| **Field naming** | YAML stays snake_case; TS domain types camelCase; map at loader boundary |
| **Template skip** | Ignore `_TEMPLATE.md` and any file not matching `*.md` resort slug pattern |

### Architecture pattern

```
content/resorts/<slug>.md
    ↓ gray-matter + Zod
ResortFrontmatter (validated)
    ↓ map to camelCase + parse body sections
Resort { slug, ...frontmatter, body: ResortBody }
    ↓ filter content_status === 'published'
getAllResorts() → Resort[]
```

### Required type coverage (PRD §8.1)

**Identity:** `nameEn`, `nameJp`, `region`, `prefecture`, `resortType`, `linkedResorts`, `tagline`, `bestFor`

**Access:** `gatewayAirport`, `airportCode`, `airportDistanceTime`, `directStation`, `transportOptions[]`, `recommendedTransport`

**Mountain:** `topElevationM`, `baseElevationM`, `verticalDropM`, `skiableAreaHa`, `numRuns`, `longestRunM`, `terrainSplit`, `steepestGradientDeg`, `lifts`, `terrainParks`, `nightSkiing`, `treeRunsOffpistePolicy`, **`trailMapUrl`**, **`trailMapNote`**

**Season (T2 labelled):** `avgAnnualSnowfallM`, `snowQuality`, `seasonTypical`, **`seasonConfirmed`**, `bestMonths`, `peakAvoid`

**Suitability:** `familyFriendly`, `snowboarderFriendly`, `englishSupport`, `skiSchool`, `onSiteRental`, `skiInSkiOut`, `adaptiveAccessibility`

**Pricing (T2):** **`liftPassAdultDay`**, `liftPassNotes`, `multiResortPass`

**Area & picks:** `nearestVillage`, `nearestTown`, `onsen`, `apresNightlife`, `localCuisine`, `nearbyDaytrips`, **`foodPicks[]`**, `accommodationPicks[]`, `rentalPicks[]`

**Media:** **`images`** — `{ hero[], card, village[], cuisine[] }` with `{ photoId, query, photographer, photographerUrl, photoUrl, alt }`

**Governance:** `contentStatus`, `lastReviewed`, `reviewedBy`, `dataConfidence`, `sources[]`

**Body (markdown):** `quickVerdict`, `courseBreakdown`, `highlights`, `watchOutFor`, `areaNarrative`, `raw`

### Zod strategy

- Use permissive types for numeric-ish fields (`z.union([z.string(), z.number()])`) — resort files mix formats
- `images.card` may be `{}` empty object — treat as optional/nullable
- `content_status`: `z.enum(['draft', 'in-review', 'published'])`
- Validation errors: log file path + Zod issue; skip file in dev with warning (don't crash entire build for one bad file)

### Warning format

```typescript
console.warn(`[content] Skipping unpublished resort: ${slug} (content_status=${status})`);
```

Use `[content]` prefix for grep-friendly build logs.

### Body section parser

Split markdown body on `## ` headers. Map:
- `Quick verdict` → `quickVerdict`
- `Course breakdown by level` → `courseBreakdown`
- `Highlights` → `highlights`
- `Watch out for` → `watchOutFor`
- `The area` → `areaNarrative`

### Previous story intelligence

- Story 1.2: Theme tokens in place; `page.tsx` is a server component placeholder — safe to import loader here
- All 20 resorts `content_status: draft` — expect `getAllResorts()` → `[]` and 20 warnings
- Do not modify `next.config.ts`, health route, or token files except minimal `page.tsx` verification hook

### Testing requirements

- **Automated:** `npm run build`, `npm run lint`
- **Manual:** Build output contains 20 `[content]` skip warnings
- **Manual:** Temporarily set one resort to `published` locally → verify count becomes 1 (optional sanity check; revert if tested)
- Co-located unit test optional — not required if build verification sufficient

### References

- [Source: epics.md § Story 1.3](../planning-artifacts/epics/phase1/epics-phase1.md)
- [Source: _powri/planning-artifacts/prds/phase1/prd-phase1.md §8.1](../planning-artifacts/prds/phase1/prd-phase1.md)
- [Source: content-model.md §4](../../content/content-model.md)
- [Source: architecture.md § Data Architecture](../planning-artifacts/architecture/phase1/architecture-phase1.md)
- [Source: Story 1.2](./1-2-design-tokens-default-theme-typography.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — pass; 20 `[content] Skipping unpublished resort` warnings (all drafts)
- `npm run lint` — pass
- `getAllResorts()` returns `[]` with current content (0 published / 20 draft)

### Completion Notes List

- Added `gray-matter` + `zod`; built `web/src/lib/content/` module (types, schema, paths, parseResortBody, loadResorts, index)
- Zod validates snake_case frontmatter; loader maps to camelCase `Resort` with full §8.1 field coverage
- Published filter + `[content]` warnings per unpublished slug; build succeeds with zero published resorts
- Home page shows published/draft counts as build-time verification hook

### File List

- `web/package.json` (modified — gray-matter, zod)
- `web/package-lock.json` (modified)
- `web/src/lib/content/types.ts` (new)
- `web/src/lib/content/schema.ts` (new)
- `web/src/lib/content/paths.ts` (new)
- `web/src/lib/content/parseResortBody.ts` (new)
- `web/src/lib/content/loadResorts.ts` (new)
- `web/src/lib/content/index.ts` (new)
- `web/src/app/page.tsx` (modified — loader verification)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve  
**Layers:** Blind Hunter, Edge Case Hunter, Acceptance Auditor

### Summary

All four ACs satisfied. gray-matter + Zod loader correctly reads sibling `content/resorts/`, maps §8.1 fields, filters published-only, and logs `[content]` warnings without failing build when zero published.

### Action Items

- [x] [Review][Defer] `getAllResorts()` re-logs warnings on every call — acceptable for build-time; memoize warnings if called per-request in later stories
- [x] [Review][Defer] No `getResortFileBySlug` for draft preview — add in Story 1.4 for `/resorts/[slug]` placeholder routes

### Review Findings

- [x] [Review][Defer] Repeated warning logs on multiple `getAllResorts()` calls — defer
- [x] [Review][Defer] Draft slug routing helper — Story 1.4
