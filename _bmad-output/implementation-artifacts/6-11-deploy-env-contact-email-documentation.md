# Story 6.11: Deploy Env — Contact Email & Env Documentation

Status: done

baseline_commit: c820d29

## Story

As a **deployer**,  
I want contact email and analytics env vars documented in one place,  
So that production Info tab and PostHog work on first deploy.

## Acceptance Criteria

1. **Given** `web/.env.example` lists `NEXT_PUBLIC_CONTACT_EMAIL`  
   **When** docs are synced  
   **Then** `docs/launch-checklist.md` §3 deploy lists all required production env vars with descriptions

2. **And** root `.env.example` cross-links to `web/.env.example` for app vars

3. **And** Info page shows real mailto when `NEXT_PUBLIC_CONTACT_EMAIL` set (existing behaviour verified)

## Tasks / Subtasks

- [x] **Launch checklist §3** — required + optional production env table + first-deploy checklist
- [x] **Root `.env.example`** — cross-link to `web/.env.example` and launch checklist
- [x] **Verify Info mailto** — `info/page.tsx` uses `getContactEmail()` → `mailto:${contactEmail}`

## Dev Agent Record

### Agent Model Used

Composer

### Verification notes

- `web/src/lib/site/appMeta.ts` reads `NEXT_PUBLIC_CONTACT_EMAIL` (fallback `contact@example.com`)
- `web/src/app/[locale]/info/page.tsx` renders `mailto:` link with env value
- Privacy/Terms pages interpolate same email via `getContactEmail()`

### File List

- `docs/launch-checklist.md`
- `.env.example`

## Senior Developer Review (AI)

**Review date:** 2026-06-16  
**Outcome:** Approve

### Review Findings

- [x] [Review][Pass] All four required production vars documented with descriptions
- [x] [Review][Pass] Root `.env.example` cross-links web vars
- [x] [Review][Pass] Info mailto wiring unchanged and correct
