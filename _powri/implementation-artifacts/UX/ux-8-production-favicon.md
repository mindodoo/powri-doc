# Story UX-8: Production favicon

Status: ready-for-dev

**Blocked:** Product owner must commit the favicon file before implementation. **No placeholder artwork.**

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

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** N/A
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** `npm run build`; confirm `/icon` or `/favicon.ico` 200 locally
- [ ] **Manual QA:** PO — production tab after deploy

## Tasks / Subtasks

- [ ] Wait for asset in repo (AC: 3)
- [ ] Place file + wire metadata if needed (AC: 1–2)
- [ ] lint + build

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

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
