# Story 5.1: Info Tab — About, Contact & Version

Status: done

baseline_commit: 48d7544

## Story

As a **user**,
I want an Info tab with product context and a way to reach the team,
So that I understand what the app is and can exercise my data rights.

## Acceptance Criteria

1. **Given** I tap Info in bottom nav  
   **When** the Info screen loads  
   **Then** About section explains product purpose in 2–3 sentences (FR-7, UX-DR18)

2. **Given** the Info screen  
   **When** rendered  
   **Then** Contact shows email for feedback and GDPR/PDPA data-rights requests

3. **Given** the Info screen  
   **When** I scroll to the bottom  
   **Then** app version string displays (from `NEXT_PUBLIC_APP_VERSION`)

4. **Given** the Info screen  
   **When** rendered  
   **Then** layout is minimal with no photography for fast load

## Tasks / Subtasks

- [x] **Site metadata helper** (AC: 2, 3)
  - [x] `getAppVersion()` + `getContactEmail()` in `lib/site/appMeta.ts`
  - [x] Document `NEXT_PUBLIC_CONTACT_EMAIL` in `.env.example`

- [x] **Info page content** (AC: 1–4)
  - [x] Replace placeholder about copy with 2–3 sentence product description
  - [x] Contact section with `mailto:` link (≥44px tap target)
  - [x] Version footer at bottom
  - [x] No photos; card layout consistent with existing Info stub

- [x] **i18n**
  - [x] Update `info.*` strings in `en.json`

- [x] **Validate**
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- **UX reference:** PRD §12.5 Screen 6 — minimal, no photography
- **Out of scope (5.2):** Privacy Policy / Terms links and `/privacy`, `/terms` pages
- **Version:** Reuse same env as analytics (`NEXT_PUBLIC_APP_VERSION`, default `1.0.0`)
- **Contact email:** `NEXT_PUBLIC_CONTACT_EMAIL` env; required for production deploy

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story 5.1]
- [Source: `_bmad-output/planning-artifacts/prd/07-features-info.md`]
- [Source: `_bmad-output/planning-artifacts/prd/13-ux-screens.md` Screen 6]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Replaced Epic 5 placeholder with About (3 sentences), Contact (`mailto:`), and version footer
- `getAppVersion()` / `getContactEmail()` read from env with sensible defaults for local dev
- Privacy/Terms links deferred to Story 5.2

### File List

- `web/.env.example` (modified)
- `web/messages/en.json` (modified — `info.*`)
- `web/src/lib/site/appMeta.ts` (new)
- `web/src/app/[locale]/info/page.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Info tab shows product about copy, GDPR/PDPA contact email with accessible mailto link, version from env, and minimal no-photo layout.

### Review Findings

- [x] [Review][Pass] About copy is 3 sentences; matches FR-7 / UX-DR18
- [x] [Review][Pass] Contact mailto uses `min-h-11` tap target + `aria-label`
- [x] [Review][Pass] Version reads `NEXT_PUBLIC_APP_VERSION` (aligned with analytics default `1.0.0`)
- [x] [Review][Defer] Privacy Policy / Terms links — Story 5.2
- [x] [Review][Defer] Set real `NEXT_PUBLIC_CONTACT_EMAIL` in Vercel before Ring 1 public share
- [x] [Review][Note] `getAppVersion()` duplicates analytics helper — acceptable; dedupe optional polish
