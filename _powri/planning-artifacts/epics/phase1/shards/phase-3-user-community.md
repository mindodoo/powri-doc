<!-- generated from epics/phase1/epics-phase1.md — do not edit; run prd-shard-regen.md -->

# Phase 3: User Community — AI Summaries (Deferred)

> **Update 2026-06-23:** Core resort reviews, comments, and moderation moved to **Phase 2 Epic 12** ([`epic-12-community-reviews.md`](../../phase2/shards/epic-12-community-reviews.md), FR-10). Auth prerequisite is **Phase 2 Epic 9** (FR-8). This shard now covers **Phase 3-only** enhancements.

**Gate:** Requires Epic 12 live with sufficient review volume before AI summarization investment.  
**Product owner requirement:** PO-5 (2026-06-14) — experience reviews on resort detail pages (**Phase 2**).

**Not Epic 6** — do not implement during Phase 1 improvements.

---

## Vision (Phase 3 delta)

After Phase 2 ships structured reviews and flat comments, Phase 3 adds **AI-generated theme summaries** (e.g. "great for powder hunters", "crowded during CNY") to complement editorial highlights/lowlights.

Individual reviews and comments are **Phase 2** — see Epic 12.

---

## Functional requirement (Phase 3 — draft)

| ID | Requirement |
|----|-------------|
| **FR-10-P3** | **Review theme summaries** — when review count ≥ threshold, display AI-generated aggregate themes on resort detail Experience section |

### Acceptance criteria (high level — refine when epic is scheduled)

**Given** a resort has ≥ N published reviews  
**When** the Experience section renders  
**Then** an optional **theme summary** block appears (expandable) alongside aggregate rating/count from Phase 2

**And** summaries are regenerated on a schedule or when review count crosses thresholds  
**And** summaries respect FR-6 cost guardrails if same AI provider  
**And** no PII from individual reviews leaks into summary text

---

## Dependencies

| Dependency | Phase | Notes |
|------------|-------|-------|
| User accounts / auth | Phase 2 Epic 9 | Supabase — done in Phase 2 |
| Reviews + comments data model | Phase 2 Epic 12 | Runtime UGC in Supabase |
| AI summarization | Phase 3 | Optional genre/themes; tie to FR-6 cost guardrails |
| Moderation tooling | Phase 2 Epic 12 | Report + admin hide; human queue optional Phase 3 |

---

## Analytics (future — tracking-plan §3)

| Event | Trigger | Key properties |
|-------|---------|----------------|
| `review_summary_expanded` | User expands AI/theme summary | `resort_slug` |

*(Phase 2 events `review_section_viewed`, `review_submitted`, etc. — see Epic 15 / tracking-plan §3.)*

---

## Implementation order (when Phase 3 community AI starts)

1. **Epic 12 live** — reviews + moderation in production
2. **Threshold tuning** — minimum review count per resort
3. **Summary job** — batch or on-demand AI when threshold met
4. **Detail UI** — expandable theme block in Experience section

---

## Phase 1 / Epic 6 exclusions

- No review UI stub on detail pages (Phase 1)
- Editorial **highlights/lowlights** remain the sole honesty signal until Epic 12 ships

---

**Related:** [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md) (AI infra) · [`epic-12-community-reviews.md`](../../phase2/shards/epic-12-community-reviews.md) (Phase 2 UGC)  
**PRD:** [`prds/phase2/prd-phase2.md`](../../../prds/phase2/prd-phase2.md) FR-10
