# Phase 2 Pre-Implementation Readiness Checklist

**Companion to:** [`prd-phase2.md`](prd-phase2.md) · **Updated:** 2026-06-23 (revision 3)

Use this checklist before opening Phase 2 implementation stories. Items marked **Done in PRD** are specified in the PRD/addendum; items marked **TODO** need a separate artifact or task.

---

## A. PRD completeness

| # | Item | Status |
|---|------|--------|
| A1 | Product scope & FR-8–17 defined | **Done in PRD** |
| A2 | Open questions OQ-2, OQ-3 closed | **Done in PRD** §11 |
| A3 | OQ-1 SEO strategy decided | **Done in PRD** §5.7 FR-14.3 |
| A4 | Internal inconsistencies resolved (journeys, glossary, photos, export) | **Done in PRD** rev 3 |
| A5 | PRD reviewer gate + `status: final` | **TODO** — after downstream docs |
| A6 | Phase 1 shard reconciliation (`11-roadmap-scope`, phase-3 epics) | **TODO** — when PRD final |

---

## B. Downstream artifacts (required before Sprint 1 code)

| # | Artifact | Owner | Status |
|---|----------|-------|--------|
| B1 | **Phase 2 architecture delta** — Supabase schema, RLS, API routes, env vars, middleware | Engineering | **Done** — [`architecture-phase2.md`](../../architecture/phase2/architecture-phase2.md) + [`supabase/migrations/001_phase2_core.sql`](../../../../supabase/migrations/001_phase2_core.sql) |
| B2 | **Phase 2 epics & stories** — P0→P1 dependency order | PM | **Done** — [`epics/phase2/epics-phase2.md`](../../epics/phase2/epics-phase2.md) + shards 09–15 |
| B3 | **Phase 2 UX spec** — Sign-in, Plan, Passport, Experience section, Map, Saved, Account | UX | **Done** — [`ux-designs/ux-phase2/`](../../ux-designs/ux-phase2/) |
| B4 | **Tracking plan §3 sync** — canonical event names from PRD §10.4 | Analytics | **Done** — [`docs/analytics/tracking-plan.md`](../../../../docs/analytics/tracking-plan.md) §3 + `phase2Events.ts` |
| B5 | **Privacy Policy + Terms** — UGC, photos, magic link, deletion | Legal/PM | **TODO** — before UGC public launch |
| B6 | **Community guidelines** — short acceptable-use for reviews/comments | PM | **TODO** — before UGC public launch |
| B7 | **Moderation runbook** — report → hide via API/Supabase | Ops | **TODO** — before UGC public launch |

---

## C. Content & data prerequisites

| # | Item | Blocks | Status |
|---|------|--------|--------|
| C1 | Add `latitude`, `longitude`, `map_default_zoom` to `content-model.md` §4 | Map FR-9 | **Done** — §4.1a |
| C2 | Populate lat/lng for all **20** published resorts | Map FR-9 | **Done** — `content/resorts/*.md` |
| C3 | Zod/build loader updated for new geo fields | Map FR-9 | **Done** — `schema.ts`, `loadResorts.ts`, launch gate |
| C4 | Badge region mapping (`badge_region` per resort) | Passport FR-11 | **Done in addendum** §C.1 |
| C5 | Badge icon assets (16) or placeholder set | Passport FR-11 | **TODO** — design; placeholders OK for dev |
| C6 | Supabase project + Vercel env vars provisioned | P0 Auth | **Done** — ref `dbeujcltupsccxxhfuji` (Tokyo); migrations applied |

**Resort geo fields spec:** see [`addendum.md`](addendum.md) §K.

---

## D. Infrastructure & env (P0 gate)

| Variable / resource | Purpose |
|---------------------|---------|
| `NEXT_PUBLIC_SUPABASE_URL` | Client auth |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Client auth |
| `SUPABASE_SERVICE_ROLE_KEY` | Server/admin only |
| `GOOGLE_PLACES_API_KEY` | Server proxy only |
| Supabase Storage buckets | `review-photos`, `passport-photos` |
| Google OAuth client | Supabase Auth provider |
| Email sender + SPF/DKIM | Magic link deliverability |

Full list in architecture doc (B1).

---

## E. Feature gates (implement in order)

| Gate | Stories allowed after |
|------|---------------------|
| **G0** | B1 + B2 complete | P0: Auth, profiles, RLS, middleware |
| **G1** | C1–C3 + B3 (map UX) | P1: Resort map |
| **G2** | B5–B7 + B3 (Experience UX) | P1: Reviews, comments, photos |
| **G3** | C4 + badge placeholders | P1: Passport + badges |
| **G4** | B3 (Plan UX) | P1: Trip skeleton |
| **G5** | B4 updated | Analytics on each shipped story |

---

## F. Launch gates (from PRD §13)

Copy of release gates — check at Phase 2 merge to `main`:

- [ ] Supabase RLS policies tested
- [ ] GDPR deletion flow E2E
- [ ] Google Places proxy rate-limited + cached
- [ ] All Phase 2 analytics events in PostHog
- [ ] Privacy Policy + Terms live
- [ ] Resort pages crawlable (SSR editorial + client UGC)
- [ ] `npm run lint && npm run build && npm run test:launch`
- [ ] Photo upload limits client + server
- [ ] Moderation report → admin hide path tested
- [ ] Google Places attribution visible on map (addendum §L)
- [ ] All 20 resorts have valid lat/lng

---

*When A5, B1–B4, and C1–C3 are complete, Phase 2 implementation may begin.*
