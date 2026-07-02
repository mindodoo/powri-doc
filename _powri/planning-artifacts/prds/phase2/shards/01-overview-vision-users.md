<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §0–2: Document Purpose, Vision & Target User

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §0–2 · **See also:** **Epics:** [`epics/phase2/epics-phase2.md`](../../../epics/phase2/epics-phase2.md) · [`INDEX.md`](../../../INDEX.md)

---

## 0. Document Purpose

This PRD defines Phase 2 of **Powri** — a mobile-first web product helping international and domestic skiers discover and plan Japan snow trips. Phase 1 (launched) ships resort discovery, detail pages, Getting There, and the Find My Resort quiz. Phase 2 adds **user accounts**, **community features**, **resort maps**, **Ski Passport gamification**, and a **trip-planning skeleton** that Phase 3 AI will populate.

**Audience:** Product, engineering, UX, analytics.  
| **Builds on:** [`prds/phase1/shards/01-overview-and-goals.md`](../../phase1/shards/01-overview-and-goals.md), [`prds/phase1/shards/02-phase1-functional-requirements.md`](../../phase1/shards/02-phase1-functional-requirements.md), [`architecture.md`](../../../architecture/phase1/architecture-phase1.md).  
**Phase 2 implementation:** [`architecture-phase2.md`](../../../architecture/phase2/architecture-phase2.md).  
**Technical depth:** [`addendum.md`](../addendum.md) (storage, auth setup, badge UI spec, AI handoff schema).  
**Pre-implementation checklist:** [`readiness-checklist.md`](../readiness-checklist.md).  
**Decision audit trail:** [`.decision-log.md`](../.decision-log.md).

---

## 1. Vision

Powri Phase 2 transforms the product from a read-only editorial guide into a **warm, community-driven planning companion**. Users can explore resorts freely (no login wall), then sign in when they want to share experiences, collect Ski Passport badges, or start building a trip.

Trip planning in Phase 2 is intentionally a **template skeleton** — not a blank form for power users. Phase 3 AI will fill the skeleton from seed content and user context; users edit the AI output. This positions Powri as an assistant that saves research time for the majority, rather than competing with spreadsheets for super-planners.

The quiz remains a **standalone fun entry point** (Phase 1 behaviour). Quiz results are not stored on user profiles in Phase 2. AI assistance (Phase 3) gets **entry-point stubs** on quiz results and resort detail pages in Phase 2, but no live chat.

**Design continuity:** Premium travel-magazine aesthetic (Theme A — warm off-white, Cormorant Garamond + DM Sans). New surfaces (Passport, Trip, Reviews) must feel like the same product, not a bolt-on social network.

---

## 2. Target User

### 2.1 Jobs To Be Done

- **Discover the right resort** — browse, filter, quiz (Phase 1; unchanged).
- **See what's around a resort** — on-mountain context plus nearby food, onsen, attractions (Phase 2 map).
- **Trust peer experiences** — read and write public reviews tied to a visible profile (Phase 2).
- **Celebrate visits** — manual check-in, optional photos, earn playful badges (Phase 2 Ski Passport).
- **Start a trip plan** — create a structured itinerary shell that AI will enrich later (Phase 2 skeleton → Phase 3 AI fill).
- **Return without friction** — persistent session on same device; login only when needed.

### 2.2 Non-Users (Phase 2)

- Users who want live booking, price comparison, or real-time inventory (out of scope).
- Users expecting AI trip generation in Phase 2 (deferred to Phase 3).
- Admin moderation workflows via dedicated admin UI (deferred to Phase 3; API-level moderation in Phase 2).

### 2.3 Key User Journeys

**UJ-1. Yuki discovers Niseko, then checks what's nearby.**  
Yuki (intermediate snowboarder, first Japan trip) browses Explore, opens Niseko United detail. She scrolls editorial content (public, no login). She taps **Map** — sees resort pin plus Google Places categories (restaurants, onsen, convenience). She taps a restaurant pin; external Google Maps opens. **Climax:** she understands the village without leaving Powri's editorial frame. **Resolution:** she taps the **heart** to save Niseko to her Saved list (stored on device as a guest); she can sign in later to sync across devices.

**UJ-2. Marcus shares his Hakuba experience after login.**  
Marcus finished a trip. On Hakuba Happo detail he taps **Write a review**. A sign-in sheet appears (Google or email — not required to browse). After auth, he submits a star rating, short text, and one photo. His display name and avatar appear on the review. **Climax:** his review is live publicly; he sees a **First Tracks** badge unlock on his Ski Passport. **Resolution:** he views his profile passport grid; optional "Plan a return trip" CTA leads to trip skeleton (UJ-4).

**UJ-3. Lin collects badges across a season.**  
Lin visits three Hokkaido resorts over winter. On each detail page she taps **I've visited** (manual check-in). She optionally uploads a lift-pass photo to her passport entry (not required). After her third Hokkaido resort she earns **Hokkaido Explorer**. After five total resorts, **Powder Hunter** unlocks. **Climax:** badge toast + passport grid update. **Resolution:** she shares her passport URL (public read) or screenshot; Slope account linking remains future/optional.

**UJ-4. Priya starts a trip skeleton for February.**  
Priya is logged in. From resort detail or nav she opens **Plan** → **New trip**. She picks one primary resort, dates, and sees a **template itinerary**: Day 1 airport → transfer → check-in; Day 2 ski; Day 3 ski + dinner placeholder; etc. Activities are editable titles/notes only — no real hotel/restaurant data yet. She saves; trip count is 3 of 5 max. **Climax:** structured multi-day view exists. **Resolution:** Phase 3 "Fill with AI" entry point visible but disabled/teased until AI ships.

**UJ-5. User reports inappropriate content.**  
Marcus (logged in) reads a review with offensive language on a resort detail page. He taps **Report** on the review, chooses a reason, and submits. The report is queued for admin review. **Climax:** he sees a confirmation — "Thank you, we'll review this." **Resolution:** an admin hides the review via API; it no longer renders. **Edge case:** guests who tap Report are prompted to sign in first — reporting requires authentication to prevent spam.

---
