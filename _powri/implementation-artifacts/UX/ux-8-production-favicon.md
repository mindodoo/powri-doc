---
baseline_commit: fa9b58dbe84cfdb2b0990a4e1bab8580dbb4dd12
---

# Story UX-8: Production favicon

Status: done

Version when done: **2.12.2.8**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a visitor with Powri in a browser tab,  
I want the Powri favicon,  
so that the tab is recognizable in production.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-8, §4).

## Acceptance Criteria

1. Production (and all environments using the same app) serve a **real favicon** from the PO-provided asset.
2. Asset lives in the Next.js App Router convention (`web/src/app/icon.png` / `favicon.ico` / `apple-icon`) **or** `web/public/` + `generateMetadata` `icons` in `web/src/app/layout.tsx`. Prefer file convention so Vercel/CDN picks it up without extra config.
3. **Do not** invent or generate a substitute mark.
4. Existing `title` / `description` metadata stay; this story is icons only.

## Testing & Definition of Done

- [x] **Unit:** N/A
- [x] **Quiz / scoring:** N/A
- [x] **Analytics:** N/A
- [x] **Content / resorts:** N/A
- [x] **Around-area labels:** N/A
- [x] **User-facing flow:** `npm run build`; confirm `/icon` or `/favicon.ico` 200 locally
- [ ] **Manual QA:** PO — production tab after deploy

## Tasks / Subtasks

- [x] Wait for asset in repo (AC: 3)
- [x] Place file + wire metadata if needed (AC: 1–2)
- [x] lint + build

## Dev Notes

**Today:** `web/src/app/layout.tsx` `generateMetadata` sets title/description only. Grep found **no** favicon/icon. Next.js 13+ App Router: `app/icon.tsx` or static `app/favicon.ico`.

**Do not:** Hotlink an external favicon. Change `NEXT_PUBLIC_APP_VERSION`.

**Unblock:** PO drops file (path noted in PR); then implement.

### Files

| Path | Role |
|------|------|
| `web/src/app/favicon.ico` or `web/src/app/icon.png` | NEW (PO bytes) |
| `web/src/app/layout.tsx` | Only if metadata `icons` required |

### References

- PRD §4 brand assets
- Next.js Metadata files: `icon`, `apple-icon`, `favicon.ico`

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story); Cursor Grok 4.6 (dev-story)

### Debug Log References

- `npm run lint`: 0 errors, 1 pre-existing warning (`supabaseStorage.ts` unused `contentType`)
- `npm run build`: pass
- Local `GET /favicon.ico`: 200, `image/x-icon`, 15406 bytes (matches `web/src/app/favicon.ico`)
- Local `GET /icon`: 404 expected — no `app/icon.png`; AC2 satisfied via `favicon.ico` file convention

### Completion Notes List

- 2026-08-21: PO-provided favicon already at `web/src/app/favicon.ico` (replaced default Next.js ico; 25931 → 15406 bytes). App Router metadata-file convention serves `/favicon.ico` without `generateMetadata.icons`. Left `layout.tsx` title/description unchanged (AC4). Did not invent artwork (AC3). Did not copy `Powri Logo/` PNGs into the app (wordmarks belong to UX-10). Did not change `NEXT_PUBLIC_APP_VERSION`.

### File List

- `_powri/implementation-artifacts/UX/ux-8-production-favicon.md`
- `_powri/implementation-artifacts/sprint-status.yaml`
- `web/src/app/favicon.ico`

### Change Log

- 2026-08-21: Production favicon wired via App Router `web/src/app/favicon.ico` (PO bytes).
- 2026-08-21: Code review complete; story approved (AC 1–4).

### Implementation Plan

Keep the PO `.ico` on the App Router metadata-file path. Do not add `icons` in `generateMetadata`. Do not generate a substitute mark.

### Review Findings

- [x] [Review][Defer] Optional `apple-icon.png` for iOS home-screen — not in AC; favicon.ico covers tab chrome [`web/src/app/`]
- [x] [Review][Defer] Optional e2e `GET /favicon.ico` 200 guard — story DoD is build + manual curl; no launch-gate hook today

### Review Findings (summary)

**Outcome: Approve** — AC 1–4 met. Single binary swap at App Router metadata path; `layout.tsx` metadata untouched.

**Dismissed (noise / intentional):** Manual QA unchecked is PO post-deploy attestation; binary diff not human-readable in PR (expected for `.ico`); `Powri Logo/` untracked folder is PO source reference, not app wiring; favicon sizes 16×16/32×32 only — PO asset, not regenerated per AC3.

### Senior Developer Review (AI)

**Review date:** 2026-08-21  
**Outcome:** Approve

#### Action Items

- [x] [Review][Defer] `apple-icon` omitted — out of UX-8 scope; add only if PO requests iOS bookmark icon
- [x] [Review][Defer] No automated favicon route test — acceptable per story DoD; optional hardening later
