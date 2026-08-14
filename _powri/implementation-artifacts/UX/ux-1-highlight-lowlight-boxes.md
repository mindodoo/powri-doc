---
baseline_commit: b22c33f2df2170b8b5cef503b41f6542dd699b84
---

# Story UX-1: Highlight / lowlight boxes + punchlines

Status: done

Version when done: **2.12.2.1**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a skier on a resort detail page,  
I want highlights and lowlights in two compact boxes with short punchlines,  
so that I can scan the verdict without a wall of bullets.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-1, D2) — see `_powri/planning-artifacts/INDEX.md`.

## Acceptance Criteria

1. Detail shows **one Highlights box** and **one Lowlights box** (not a two-column chip list with per-item colored pills).
2. **Two boxed containers** with color-coded chrome per variant: highlights use `bg-highlight-bg` + `text-accent` + `CircleCheck`; lowlights use `bg-lowlight-bg` + `text-accent-warm` + `AlertTriangle`. Bullets inside boxes use neutral `text-text-primary` (not per-item colored pills). Differentiation is box-level heading/icon color, not per-bullet chips.
3. Each bullet in `content/resorts/*.md` **Highlights** and **Watch out for** is rewritten as a **short punchline** (keep meaning; no CSS truncation). Engineering owns the rewrite (D2). Skip `_TEMPLATE.md` editorial examples only if they are not loaded as a live resort.
4. Empty side: if a resort has no highlights or no lowlights, omit that box (same as today when a list is empty).
5. `parseBulletList` still feeds the UI — keep `- ` markdown bullets.

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** N/A unless `parseBulletList` / `detailSections` mapping changes
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A
- [x] **Content / resorts:** `npm run test:launch` (content still parses)
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** Manual on one flagship + one sparse resort; Playwright not required unless an existing detail spec asserts old layout
- [x] **Manual QA:** PO — punchline quality + equal box chrome

## Tasks / Subtasks

- [x] Restyle `HighlightLowlightRow.tsx` into two boxes (AC: 1–2, 4)
- [x] Rewrite Highlights / Watch out for in all live `content/resorts/*.md` (AC: 3)
- [x] `npm run lint && npm run build && npm run test:launch`

## Dev Notes

**Today:** `HighlightLowlightRow` is a `grid-cols-2` of columns; each item is a rounded chip; highlights use `bg-highlight-bg` + `CircleCheck` + `text-accent`; lowlights use `bg-lowlight-bg` + `AlertTriangle` + `text-accent-warm` (Story 6.6). Wired from `ResortDetailContent` via `toResortDetailSectionsData` → `parseBulletList(resort.body.highlights | watchOutFor)`.

**Change:** Two bordered boxes (not per-bullet chips). Keep color-coded box chrome: highlights `bg-highlight-bg` + `text-accent`; lowlights `bg-lowlight-bg` + `text-accent-warm`. Bullet text neutral inside boxes. Stacked on mobile; may sit side-by-side on desktop if both exist.

**Do not:** Truncate long strings with `line-clamp`. Invent new content facts. Touch quiz or map POI `highlight` fields.

**Copy bar:** Punchlines ≈ one short clause (Furano today: “Dry powder with frequent bluebird days” is already close; “Right beside Furano town — lots of lodging and dining” → e.g. “Town at the base”). Keep comparable count per resort.

### Files

| Path | Role |
|------|------|
| `web/src/components/resort/HighlightLowlightRow.tsx` | UPDATE layout/tokens |
| `web/src/components/resort/ResortDetailContent.tsx` | Keep wiring; heading ids |
| `content/resorts/*.md` | UPDATE `## Highlights` / `## Watch out for` |
| `web/src/lib/content/parseBulletList.ts` | Preserve |
| `web/messages/en.json` `resortDetail.highlights*` | Labels only |

### References

- PRD UX-1, D2
- Prior: Epic 6.6 header icons — color-coded box headings retained; per-bullet chips removed

## Dev Agent Record

### Agent Model Used

Composer (dev-story)

### Debug Log References

- `npm run lint` — pass (pre-existing warnings only)
- `npm run build` — pass; all 20 resort pages static-generate
- `npm run test:launch` — content gates pass; exits non-zero on pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env requirement (not introduced by this story)

### Completion Notes List

- Replaced per-bullet colored chips with two bordered `VerdictBox` containers; box-level color coding retained (`bg-highlight-bg` / `text-accent` vs `bg-lowlight-bg` / `text-accent-warm`); bullet text neutral (`text-text-primary`).
- Empty lists omit their box (no placeholder column).
- Rewrote Highlights / Watch out for punchlines in all 20 live resort markdown files; `_TEMPLATE.md` unchanged.
- Layout: stacked on mobile, `md:grid-cols-2` when both boxes present.
- Code review: `SectionHeader` `headingId` wired for season, highlights, and terrain sections in `ResortDetailContent.tsx`.

### File List

- `web/src/components/resort/HighlightLowlightRow.tsx`
- `content/resorts/appi-kogen.md`
- `content/resorts/furano.md`
- `content/resorts/gala-yuzawa.md`
- `content/resorts/hakuba-valley.md`
- `content/resorts/joetsu-kokusai.md`
- `content/resorts/kandatsu-kogen.md`
- `content/resorts/kiroro.md`
- `content/resorts/lotte-arai.md`
- `content/resorts/madarao-kogen.md`
- `content/resorts/myoko-kogen.md`
- `content/resorts/naeba.md`
- `content/resorts/nekoma-alts-bandai.md`
- `content/resorts/niseko-united.md`
- `content/resorts/nozawa-onsen.md`
- `content/resorts/rusutsu.md`
- `content/resorts/sahoro.md`
- `content/resorts/shiga-kogen.md`
- `content/resorts/tomamu.md`
- `content/resorts/tsugaike-kogen.md`
- `content/resorts/zao-onsen.md`
- `web/src/components/resort/ResortDetailContent.tsx`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `_powri/implementation-artifacts/UX/ux-1-highlight-lowlight-boxes.md`

## Change Log

- 2026-08-14 — UX-1: highlight/lowlight boxes + punchline rewrites (Composer dev-story)
- 2026-08-14 — Code review: AC #2 clarified (color-coded boxes); `appMeta` revert; `headingId` a11y fix (Composer)

### Review Findings

- [x] [Review][Decision] Box chrome color-coded — **resolved:** keep accent tokens per PO; AC #2 updated to match intent (box-level color, not per-bullet chips).
- [x] [Review][Patch] Unify `VerdictBox` tokens — **dismissed:** PO chose to keep color-coded box chrome.
- [x] [Review][Patch] Revert out-of-scope `appMeta.ts` version bump — **fixed.**
- [x] [Review][Defer] `aria-labelledby` / `headingId` mismatch — **fixed** in `ResortDetailContent.tsx` (season, highlights, terrain).
