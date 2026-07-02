<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §3–4: Glossary & Feature Priority

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §3–4 · **See also:** **Build sequence:** P0 auth → P1 features → P2 AI stubs · [`INDEX.md`](../../../INDEX.md)

---

## 3. Glossary

| Term | Definition |
|------|------------|
| **Powri** | Product brand (Japan ski/snowboard trip planner web app). |
| **Resort** | One of 20 seed resorts; content from `content/resorts/<slug>.md`; always public. |
| **Guest** | Unauthenticated visitor; browse, quiz, view public reviews/comments, save resorts locally (device only). |
| **User** | Authenticated account; review, comment, trips, passport, synced saves, report content. |
| **Admin** | User with `role = admin` on profile; hide/delete UGC via API/DB (UI Phase 3). |
| **Public profile** | Read-only page at `/u/:username` showing display name, badges, passport grid, and public reviews. |
| **Ski Passport** | User's gamified visit log + badge collection (Hall of Fame). |
| **Check-in** | Manual "I've visited this resort" action; no evidence required. |
| **Trip** | User-owned itinerary object (max 5 per user); template activities in Phase 2. |
| **Activity** | Single item in a trip day (airport, transport, accommodation, ski, restaurant, attraction, custom). |
| **Template itinerary** | Pre-structured day/activity placeholders — not live POI/booking data. |
| **Seed content** | Build-time markdown resort data (T1/T2 tiers). |
| **UGC** | User-generated content: reviews, comments, passport photos. |
| **Saved resort** | User-bookmarked resort via heart icon; guest = device-local, user = synced. |
| **AI handoff** | Phase 2 data shapes and entry points Phase 3 AI consumes to generate trip content. |

---

## 4. Feature Priority & Build Sequence

Phase 2 is dependency-ordered. **Do not start community features without auth + database.**

| Priority | Epic theme | Rationale |
|----------|------------|-----------|
| **P0** | Auth + DB + roles + GDPR account flows | Foundation for all logged-in features |
| **P0** | SEO guardrails (public resort content) | Non-negotiable; embed in every story |
| **P1** | Resort map (Google Places) | High discovery value; low user friction; no login |
| **P1** | Reviews + comments + photos + moderation API | Core community; photos ship with reviews (not deferred) |
| **P1** | Ski Passport + badges | Engagement loop; complements reviews |
| **P1** | Trip skeleton UI + schema | Phase 3 AI dependency |
| **P1** | Saved resorts (heart) | Lightweight personalisation; Phase 3 AI context |
| **P1** | Public profile (`/u/:username`) | Badges, passport, reviews showcase |
| **P1** | Adaptive navigation (mobile bottom + desktop top) | Phase 2 surfaces need IA without crowding |
| **P2** | AI entry-point stubs (disabled) | Prepare Phase 3 without API cost |
| **P3** | Admin moderation UI | Deferred Phase 3; API sufficient at low traffic |
| **P3** | Review aggregate themes / AI summary | Phase 3 with FR-6 |
| **Future** | Slope account linking | Partnership TBD |

### Deferred / optional (not Phase 2 MVP)

| Feature | When |
|---------|------|
| **Notification opt-in (email)** | Phase 2.1 or Phase 3 |
| **Share trip skeleton (read-only link)** | Phase 2.1 |

---
