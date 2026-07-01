<!-- generated from epics/phase1/epics-phase1.md — do not edit; run prd-shard-regen.md -->

# Epic 5: Info, Legal & Launch Readiness

**Stories:** 5.1–5.5 · **FR:** FR-7 · **NFR:** NFR-13 · **PRD:** [`prds/phase1/shards/07-features-info.md`](../../../prds/phase1/shards/07-features-info.md)

Info tab, legal pages, satisfaction survey, analytics verification, launch checklist.

---

## Story 5.1: Info Tab — About, Contact & Version

About 2–3 sentences; contact email for GDPR; app version; minimal layout, no photos (UX-DR18).

## Story 5.2: Privacy Policy & Terms of Service Pages

`/privacy`, `/terms`; analytics + third-party disclosure; "information as-is" disclaimer.

## Story 5.3: Satisfaction Micro-Survey

After 2nd distinct resort detail in session; 1–5 stars; `satisfaction_prompt_submitted`; once per session.

## Story 5.4: Phase 1 Analytics Verification & Tracking Plan Sync

E2E test pass fires all tracking-plan §2 Phase 1 events (excludes AI chat — Phase 3); sync tracking-plan on changes; consent gating.

## Story 5.5: Launch Gate Checklist (Content & QA)

20 published resorts; 375/430px QA; broken links; Unsplash site-wide; counter-metrics in PostHog; production smoke.

**Checklist:** [`docs/qa/phase1/launch-checklist.md`](../../../../../docs/qa/phase1/launch-checklist.md) · **Script:** `web/scripts/verify-launch-gates.ts` (`npm run test:launch`)

---

**Full ACs:** [`epics/phase1/epics-phase1.md`](../epics-phase1.md) Epic 5 section  
**Gates:** [`prds/phase1/shards/11-roadmap-scope.md`](../../../prds/phase1/shards/11-roadmap-scope.md)
