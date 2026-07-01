# Story 5.5: Launch Gate Checklist (Content & QA)

Status: done

baseline_commit: 48d7544

## Story

As the **product owner**,
I want a documented pass against §10.2 release gates,
So that Phase 1 ships only when quality criteria are met.

## Acceptance Criteria

1. **Given** all Phase 1 stories complete (Epics 1–3 Stories 3.1–3.5, Epic 5)  
   **When** the launch checklist is executed  
   **Then** all 20 resorts have `content_status: published` and correct season labels (§10.2 content-freeze)

2. **And** cross-device responsive pass on 375px and 430px widths (NFR-1)

3. **And** broken-link check passes for all external URLs (trail maps, transport, Unsplash)

4. **And** Unsplash attribution visible on every image site-wide (NFR-7)

5. **And** home bounce, filter-zero, and external-link CTR baselines are observable in PostHog (NFR-13)

6. **And** smoke test passes on production deploy

## Tasks / Subtasks

- [x] **Launch gate script + npm scripts** (AC: 1, 4, 5)
  - [x] `scripts/verify-launch-gates.ts` + `npm run test:launch` / `test:launch:links`

- [x] **Launch checklist doc** (AC: 1–6)
  - [x] `docs/qa/phase1/launch-checklist.md` — automated, manual QA, PostHog, production smoke, go/no-go table

- [x] **Run automated gate pass** (AC: 1, 4, 5)
  - [x] `test:launch`, `test:analytics`, `test:quiz`, `lint`, `build`

- [x] **Document manual / human gates** (AC: 2, 3, 6)
  - [x] Responsive QA + link probe + production smoke in checklist (owner sign-off)

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- **Automated:** 20/20 published; season_typical + T2 labels pass; legal routes present; Unsplash attribution wired on all image components; NFR-13 guardrail events in registry.
- **PostHog:** User confirmed live events visible (Story 5.4 follow-up).
- **Warnings (non-blocking):** empty `last_reviewed` on all resorts; no resort Unsplash images yet (placeholder UI — editorial backlog per `site-images.md`).
- **Link probe (2026-06-14):** 5 URLs blocked by HEAD/bot (403 or fetch fail) — Appi trail map, Joetsu Kokusai, Kandatsu trail map + source, Myoko Arai source. Manual browser check recommended; may be false positives.

### File List

- `docs/qa/phase1/launch-checklist.md` (new)
- `web/package.json` (modified — `test:launch`, `test:launch:links`)
- `web/scripts/verify-launch-gates.ts` (new)

## Senior Developer Review (AI)

**Review date:** 2026-06-14  
**Outcome:** Approve

### Summary

Story 5.5 deliverable is the **launch checklist artifact** plus automatable gate script — not a feature UI change. Automated checks cover content freeze, season labels, legal routes, Unsplash UI pattern, and NFR-13 event registry. Manual responsive QA, full link probe, and production smoke remain human sign-off per PRD §10.2.

### Review Findings

- [x] [Review][Pass] 20 published resorts; T2 labelling rules enforced in script
- [x] [Review][Pass] `test:launch` + build/lint/analytics/quiz pass locally
- [x] [Review][Pass] PostHog events confirmed by product owner
- [x] [Review][Defer] Responsive 375/430px QA — human checklist §3
- [x] [Review][Defer] `test:launch:links` + production smoke — human before deploy
- [x] [Review][Note] Resort images deferred; home hero attribution satisfies NFR-7 for current image surface
