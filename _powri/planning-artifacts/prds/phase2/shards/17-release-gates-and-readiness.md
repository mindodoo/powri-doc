<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §13–14: Release Gates & Downstream Artifacts

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §13–14 · **See also:** **Checklist:** [`readiness-checklist.md`](../readiness-checklist.md) · [`INDEX.md`](../../../INDEX.md)

---

## 13. Release Gates (Phase 2)

- [ ] Pre-implementation checklist complete ([`readiness-checklist.md`](../readiness-checklist.md) §A–D)
- [ ] All 20 resorts have `latitude` / `longitude` in seed content
- [ ] Supabase RLS policies tested (user cannot read others' draft trips)
- [ ] GDPR deletion flow end-to-end tested
- [ ] Google Places proxy rate-limited and cached
- [ ] Google Places attribution visible on map UI
- [ ] All new analytics events verified in PostHog (`docs/analytics/tracking-plan.md` synced)
- [ ] Privacy Policy + Terms + community guidelines updated for UGC
- [ ] Resort pages remain crawlable (SSR/SSG editorial body)
- [ ] `npm run lint && npm run build && npm run test:launch` pass
- [ ] Photo upload limits enforced client + server
- [ ] Moderation report → admin hide path tested manually

---
## 14. Downstream Artifacts Required Before Implementation

Do **not** start Sprint 1 code until these exist. Full checklist: [`readiness-checklist.md`](../readiness-checklist.md).

| Artifact | Skill / action | Blocks |
|----------|--------------|--------|
| Phase 2 architecture delta | `bmad-create-architecture` | P0 auth, all API routes |
| Phase 2 epics & stories | `bmad-create-epics-and-stories` | All implementation |
| Phase 2 UX spec (screens) | `bmad-ux` | UI stories |
| Tracking plan §3 update | Edit `docs/analytics/tracking-plan.md` | Analytics per story |
| Content schema + lat/lng | Edit `content-model.md` + 20 resorts | Map FR-9 |
| Privacy / Terms / guidelines | Legal + PM | UGC public launch |

---

*PRD revision 3 patches inconsistencies and closes OQ-1–3. Next: complete [`readiness-checklist.md`](../readiness-checklist.md) §B–D, then `bmad-create-architecture` → `bmad-create-epics-and-stories`.*
