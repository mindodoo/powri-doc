<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §8–9: Non-Goals & MVP Scope

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §8–9 · **See also:** **Readiness:** [`readiness-checklist.md`](../readiness-checklist.md) · [`INDEX.md`](../../../INDEX.md)

---

## 8. Non-Goals (Explicit)

- Apple Sign-In (Phase 2) — Google + email only to reduce setup friction.
- Anonymous reviews or comments.
- Storing quiz results on user profile (Phase 2).
- Live AI chat or trip generation (Phase 3).
- Admin moderation UI (Phase 3).
- Real accommodation/restaurant inventory in trips (Phase 3 AI + optional integrations).
- Slope partnership / account linking (future).
- Monetisation / affiliate (unchanged deferral).
- TH/JA full locales (scaffold only).
- Evidence-based passport check-in.

---

## 9. MVP Scope (Phase 2)

### 9.1 In Scope

- Supabase auth (Google + email), roles, GDPR deletion/export
- Resort map with Google Places proxy + cache
- Reviews, comments, report, admin API moderation
- Ski Passport check-in, badges, optional photos
- Trip skeleton (5 max), template itinerary, edit UI
- Saved resorts (heart + `/saved` list; guest localStorage + account sync)
- Adaptive navigation (4-tab mobile bottom nav + desktop top nav)
- Public profile (`/u/:username`) + account settings
- AI entry-point stubs (disabled)
- SEO public content guardrails
- Analytics events (see §10)
- Privacy/Terms update for UGC + accounts

### 9.2 Out of Scope for Phase 2

| Item | Reason / When |
|------|----------------|
| AI trip fill | Phase 3 |
| Admin UI | Phase 3 — low traffic; API/DB moderation sufficient |
| Review AI summaries | Phase 3 |
| Apple auth | PO decision — cost/complexity |
| Multi-resort trips | Simplify Phase 2 skeleton |
| Email notifications | Optional P2 — not committed |

---
