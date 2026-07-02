<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §10–12: Success Metrics, Decisions & Assumptions

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §10–12 · **See also:** **Tracking plan:** [`docs/analytics/tracking-plan.md`](../../../../docs/analytics/tracking-plan.md) · [`INDEX.md`](../../../INDEX.md)

---

## 10. Success Metrics

### 10.1 Business Success Criteria

| ID | Metric | Target (90 days post Phase 2 launch) | Notes |
|----|--------|--------------------------------------|-------|
| **BS-1** | Registered users | ≥ 500 | Organic + quiz funnel |
| **BS-2** | Weekly active users (WAU) | ≥ 30% of registered | Return intent |
| **BS-3** | UGC participation rate | ≥ 15% of WAU submit ≥1 review or check-in | Community health |
| **BS-4** | Trip skeleton creation | ≥ 20% of registered users create ≥1 trip | Phase 3 AI funnel |
| **BS-5** | SEO stability | Organic resort page traffic ≥ 90% of Phase 1 baseline | FR-14 guardrail |
| **BS-6** | Account deletion SLA | 100% within 30 days | Compliance |

### 10.2 Feature Analytics Success Criteria

| ID | Metric | Target | Validates |
|----|--------|--------|-----------|
| **SM-1** | Map engagement | ≥ 25% of `resort_detail_viewed` also fire `map_section_viewed` | FR-9 |
| **SM-2** | Review submission rate | ≥ 5% of logged-in resort viewers submit review | FR-10 |
| **SM-3** | Badge earn rate | ≥ 40% of users with ≥1 check-in earn ≥2 badges | FR-11 |
| **SM-4** | Trip quota usage | Average 1.5 trips/user among trip creators | FR-12 |
| **SM-5** | AI stub tap-through | ≥ 10% of stub taps → sign-in or trip create | FR-13 Phase 3 funnel |
| **SM-6** | Auth conversion | ≥ 35% of sign-in sheet opens complete auth | FR-8 |
| **SM-7** | Report rate | < 2% of reviews reported (quality signal) | FR-10 |
| **SM-8** | Saved resort adoption | ≥ 20% of WAU save ≥1 resort | FR-15 |
| **SM-9** | Magic-link completion | ≥ 30% of magic-link sends → click within 15 min | FR-8 |

### 10.3 Counter-Metrics (do not optimize)

| ID | Metric | Why |
|----|--------|-----|
| **SM-C1** | Raw sign-up count without WAU | Vanity metric — prefer BS-2 |
| **SM-C2** | Check-ins without visits | Discourage fake badge farming — monitor anomalies |
| **SM-C3** | Trip count without edit events | Empty templates don't predict Phase 3 AI success |

### 10.4 New Analytics Events (add to tracking-plan)

| Event | Trigger | Key properties |
|-------|---------|----------------|
| `auth_sign_in_started` | Sign-in sheet opened | `provider` (`google`/`email`), `trigger` (`review`/`trip`/`passport`/`nav`) |
| `auth_sign_in_completed` | Successful auth | `provider`, `is_new_user` |
| `auth_sign_out` | User signs out | — |
| `account_deletion_requested` | User requests deletion | — |
| `map_section_viewed` | Map enters viewport | `resort_slug` |
| `map_pin_tapped` | Place pin tapped | `resort_slug`, `place_category` |
| `review_submitted` | Review posted | `resort_slug`, `rating`, `photo_count` |
| `comment_submitted` | Comment posted | `resort_slug` |
| `content_reported` | Report filed | `content_type` (`review`/`comment`), `reason` |
| `passport_check_in` | Manual visit recorded | `resort_slug` |
| `badge_earned` | Badge unlocked | `badge_id` |
| `trip_created` | New trip saved | `resort_slug`, `day_count` |
| `trip_deleted` | Trip removed | `trip_count_remaining` |
| `trip_activity_edited` | Activity title/notes changed | `activity_type` |
| `ai_entry_point_tapped` | AI stub clicked | `surface`, `ai_available` |
| `resort_saved` | Heart toggled on | `resort_slug`, `surface` (`card`/`detail`), `is_logged_in` |
| `resort_unsaved` | Heart toggled off | `resort_slug`, `surface`, `is_logged_in` |
| `saved_list_viewed` | `/saved` page opened | `saved_count`, `is_logged_in` |
| `magic_link_sent` | Email link requested | — |
| `magic_link_completed` | User lands from email link | `is_new_user` |
| `nav_destination_tapped` | Nav item tapped | `destination`, `nav_type` (`bottom`/`top`/`hamburger`) |

---

## 11. Resolved Decisions (formerly open questions)

| # | Question | Decision | Reference |
|---|----------|----------|-----------|
| ~~OQ-1~~ | Reviews indexable by Google — full text or aggregate? | **Aggregate rating + count in SSR HTML; review bodies client-hydrated; no per-review URLs; profiles `noindex`** | FR-14.3 |
| ~~OQ-2~~ | Flat comments vs one-level replies? | **Flat comment thread only — no nesting in Phase 2** | FR-10.2 |
| ~~OQ-3~~ | Desktop breakpoint 768 vs 1024? | **`md` = 768px** — hide bottom nav, show top nav | FR-16.2 |

### Remaining open items (non-blocking for PRD final)

| # | Question | Owner | Revisit |
|---|----------|-------|---------|
| OQ-4 | Review author anonymization on account delete — legal confirm | Legal | Before UGC launch |
| OQ-5 | Public profile `noindex` lift threshold (e.g. N reviews site-wide) | PM + SEO | Phase 2.1 |

---

## 12. Assumptions Index

- Email auth: magic link only (no password); Google OAuth as parallel option.
- Magic-link trust UX follows addendum §A.1 copy patterns.
- Navigation: 4-tab mobile bottom nav + desktop top nav at **768px** + hamburger overflow (FR-16).
- Saved resorts: guest localStorage + merge on login.
- Session: **30-day rolling idle**, **90-day absolute** max (FR-8.1).
- Single-resort trips only in Phase 2.
- **Flat comments only** — no nested replies (OQ-2 closed).
- **SEO:** aggregate in SSR, review bodies client-side (OQ-1 closed).
- Logged-in required to report content.
- Google Places free tier sufficient at 20-resort scale.
- Map render: Leaflet + OSM tiles; POIs from Google Places proxy.
- Admin role: manual `profiles.role` assignment only (FR-8.6).
- Profanity: server-side blocklist on display names (addendum §O).
- Badge regions: derived from seed content `region` field (addendum §C.1).
- Ski season for badges: **1 Nov – 30 Apr JST** (addendum §C.2).
- Admin moderation via API/Supabase dashboard until Phase 3 UI.

---
