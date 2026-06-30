# Requirements Inventory

**Source:** Phase 1 [`../epics.md`](../epics.md) · Phase 2 [`epics-phase2.md`](epics-phase2.md)  
**PRD:** Phase 1 [`../prd/02-phase1-functional-requirements.md`](../prd/02-phase1-functional-requirements.md) · Phase 2 [`../prds/prd-japan_winter_sport-2026-06-23/prd.md`](../prds/prd-japan_winter_sport-2026-06-23/prd.md)

Phase 1 — all 20 resorts, **Home · Explore · Info**, **no AI assistance**, no Plan tab. **FR-6 → Phase 3:** [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md).

Phase 2 — auth, saved resorts, map, reviews, passport, trip skeleton, public profile. **Full epics:** [`epics-phase2.md`](epics-phase2.md).

---

## Functional Requirements — Phase 1

| ID | Summary |
|----|---------|
| **FR-1** | 20 resort cards; filter chips + search without reload; card → detail with `resort_slug`; Explore reuses same data |
| **FR-2** | Detail: hero, verdict, highlights/lowlights, terrain, season-labelled T2, `trail_map_url` link; Unsplash attribution |
| **FR-3** | Getting There: ≥2 transport options, gateway ≥3 experiences, collapsible practical tips |
| **FR-4** | 4-question quiz; Serious/Cosmic toggle; shared scoring; Cosmic results reveal; `quiz_started` / `quiz_completed` + `quiz_mode` |
| **FR-5** | ≤4 onboarding cards; skip + get started; local flag; no photos |
| **FR-6** | AI chat from detail; seed content first; §4.4 guardrails; chat events — **Phase 3 only** |
| **FR-7** | Info tab: Privacy, Terms, about, version, contact email |

---

## Functional Requirements — Phase 2

| ID | Summary |
|----|---------|
| **FR-8** | Google OAuth + magic link auth; guest browse unchanged; profile basics; GDPR export/deletion; user/admin roles |
| **FR-9** | Resort map section (Leaflet + OSM); Google Places POIs via server proxy; lazy-loaded; cost controls |
| **FR-10** | Structured reviews + flat comments; report flow; admin hide via API |
| **FR-11** | Manual check-in; optional passport photo; badge awards; passport view |
| **FR-12** | Trip CRUD (max 5); template skeleton itinerary; list/detail views; AI handoff hooks (Phase 3) |
| **FR-13** | Phase 3 AI entry stubs on resort detail + quiz results |
| **FR-14** | Public resort SEO unchanged; auth-gated surfaces only; SSR aggregate rating, client-hydrated review bodies |
| **FR-15** | Save/unsave resort (guest localStorage → DB merge on login); `/saved` list |
| **FR-16** | Adaptive nav: mobile bottom **Home · Explore · Saved · Plan**; desktop top nav; hamburger overflow |
| **FR-17** | Public profile `/u/:username`; account settings; `noindex` on profiles |

---

## Non-Functional Requirements — Phase 1 (NFR-1–14)

| ID | Summary |
|----|---------|
| NFR-1 | Mobile 375–430px; bottom nav always visible |
| NFR-2 | SSG/ISR; fast onboarding/Info; all 20 reachable ≤2 taps |
| NFR-3 | CSP, SRI, no inline scripts, sanitize inputs, server-side API keys, rate limits |
| NFR-4 | GDPR consent; anonymous default; no PII in events |
| NFR-5 | Markdown build-time import; `content_status: published` filter |
| NFR-6 | T2 season labels + last-reviewed |
| NFR-7 | Unsplash attribution component on every image |
| NFR-8 | All Phase 1 events in tracking-plan; verify in PostHog |
| NFR-9 | AI caps: 10 turns, token limits, IP daily cap, kill-switch |
| NFR-10 | Contrast, ≥44px tap targets, semantic HTML |
| NFR-11 | `npm audit` clean; lock file committed |
| NFR-12 | Clean outbound links; `noopener noreferrer` |
| NFR-13 | Counter-metrics: bounce, filter_zero, external CTR, satisfaction survey |
| NFR-14 | EN only Phase 1; i18n scaffold |

---

## Non-Functional Requirements — Phase 2 (NFR-10–17 per Phase 2 PRD §6)

Phase 2 PRD extends numbering from NFR-10; all Phase 1 NFRs remain in force unless superseded.

| ID | Summary |
|----|---------|
| NFR-10 | UGC sanitized on input; HTML stripped; DOMPurify on render |
| NFR-11 | Supabase RLS: users read/write own data; public read on published reviews |
| NFR-12 | Photo upload MIME validation; max 5/review, 1/check-in; 10 MB; WebP/JPEG/PNG |
| NFR-13 | Session idle 30d + absolute 90d; secure cookie flags |
| NFR-14 | Account deletion within 30 days; immediate session revocation |
| NFR-15 | Google Places + Supabase keys server-side only |
| NFR-16 | CWV on resort detail with map — no regression vs Phase 1 |
| NFR-17 | Adaptive nav per FR-16 |

---

## UX Design Requirements (UX-DR1–18)

| ID | Component / pattern |
|----|---------------------|
| UX-DR1 | Theme A design tokens |
| UX-DR2 | Cormorant Garamond + DM Sans |
| UX-DR3 | `BottomNav` (Phase 1: Home · Explore · Info; Phase 2: Home · Explore · Saved · Plan) |
| UX-DR4 | `HeroCarousel` |
| UX-DR5 | `ResortCard` + shared-element transition |
| UX-DR6 | `FilterChip` |
| UX-DR7 | `QuickVerdict` |
| UX-DR8 | `HighlightLowlightRow` |
| UX-DR9 | `StatRow` |
| UX-DR10 | `TransportRow` |
| UX-DR11 | `OnboardingCard` |
| UX-DR12 | `ResortQuizCard` — Serious/Cosmic copy variants, shared enums |
| UX-DR13 | `ChatBubble` |
| UX-DR14 | `UnsplashAttribution` |
| UX-DR15 | `SectionHeader` |
| UX-DR16 | Sticky Home top bar + inline search |
| UX-DR17 | Floating "Plan with AI →" |
| UX-DR18 | Info screen minimal |

---

## FR Coverage Map

### Phase 1

| FR | Epic | Stories |
|----|------|---------|
| FR-1 | Epic 2 | 2.2–2.6 |
| FR-4 | Epic 2 | 2.7 |
| FR-5 | Epic 2 | 2.1 |
| FR-2 | Epic 3 | 3.1–3.3 |
| FR-3 | Epic 3 | 3.4–3.5 |
| FR-6 | **Phase 3** | 3.6 + 4.1–4.4 |
| FR-7 | Epic 5 | 5.1–5.2 |

### Phase 2

| FR | Epic | Stories |
|----|------|---------|
| FR-8 | Epic 9 | 9.1–9.5 |
| FR-16 | Epic 10 | 10.1–10.2 |
| FR-15 | Epic 10 | 10.3–10.4 |
| FR-9 | Epic 11 | 11.1–11.3 |
| FR-10 | Epic 12 | 12.1–12.4 |
| FR-11 | Epic 13 | 13.1–13.2 |
| FR-17 | Epic 13 | 13.3–13.4 |
| FR-12 | Epic 14 | 14.1–14.3 |
| FR-13 | Epic 15 | 15.1 |
| FR-14 | Epic 15 | 15.4 |

**Cross-cutting:** NFR-3–5, NFR-8, UX-DR1–2 from Epic 1; NFR-7 Epic 3; NFR-9 Phase 3 (Epic 4); NFR-13 Epic 5; Phase 2 NFR-10–17 across Epics 9–15.

---

## Additional stack notes

**Phase 1:** Next.js + Tailwind + Zustand; gray-matter loader; serverless AI proxy; PostHog EU + Plausible; `content/site-images.md` for hero; all 20 published before launch.

**Phase 2:** Supabase PostgreSQL + Storage + Auth; Leaflet + OSM; Google Places server proxy; HTTP-only session cookies via Supabase SSR.

**Architecture:** [`../architecture.md`](../architecture.md) · Phase 2 delta [`../architecture-phase2.md`](../architecture-phase2.md)
