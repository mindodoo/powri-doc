# Requirements Inventory

**Source:** [`../epics.md`](../epics.md) · **PRD FR summary:** [`../prd/02-phase1-functional-requirements.md`](../prd/02-phase1-functional-requirements.md)

Phase 1 only — all 20 resorts, **Home · Explore · Info**, **no AI assistance**, no Plan tab, no live snow, no accommodation map. **FR-6 → Phase 3:** [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md).

---

## Functional Requirements

| ID | Summary |
|----|---------|
| **FR-1** | 20 resort cards; filter chips + search without reload; card → detail with `resort_slug`; Explore reuses same data |
| **FR-2** | Detail: hero, verdict, highlights/lowlights, terrain, season-labelled T2, `trail_map_url` link; Unsplash attribution |
| **FR-3** | Getting There: ≥2 transport options, gateway ≥3 experiences, collapsible practical tips |
| **FR-4** | 4-question quiz; Serious/Cosmic toggle; shared scoring; **Cosmic results reveal** (interstitial + header + subline); `quiz_started` / `quiz_completed` + `quiz_mode` |
| **FR-5** | ≤4 onboarding cards; skip + get started; local flag; no photos |
| **FR-6** | AI chat from detail; seed content first; §4.4 guardrails; chat events — **Phase 3 only** |
| **FR-7** | Info tab: Privacy, Terms, about, version, contact email |

---

## Non-Functional Requirements (NFR-1–14)

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

## UX Design Requirements (UX-DR1–18)

| ID | Component / pattern |
|----|---------------------|
| UX-DR1 | Theme A design tokens |
| UX-DR2 | Cormorant Garamond + DM Sans |
| UX-DR3 | `BottomNav` (Home · Explore · Info) |
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

| FR | Epic | Stories |
|----|------|---------|
| FR-1 | Epic 2 | 2.2–2.6 |
| FR-4 | Epic 2 | 2.7 |
| FR-5 | Epic 2 | 2.1 |
| FR-2 | Epic 3 | 3.1–3.3 |
| FR-3 | Epic 3 | 3.4–3.5 |
| FR-6 | **Phase 3** | 3.6 + 4.1–4.4 |
| FR-7 | Epic 5 | 5.1–5.2 |

**Cross-cutting:** NFR-3–5, NFR-8, UX-DR1–2 from Epic 1; NFR-7 Epic 3; NFR-9 Phase 3 (Epic 4); NFR-13 Epic 5.

---

## Additional stack notes

Next.js + Tailwind + Zustand; gray-matter loader; serverless AI proxy; PostHog EU + Plausible; `content/site-images.md` for hero; no monetisation Phase 1; all 20 published before launch.

**Architecture detail:** [`../architecture.md`](../architecture.md)
