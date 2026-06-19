# Phase 3: User Community & Resort Reviews (Deferred)

**Gate:** Requires **user accounts / login** (Phase 2+) and product traction before UGC moderation investment.  
**Product owner requirement:** PO-5 (2026-06-14) — experience reviews on resort detail pages.

**Not Epic 6** — do not implement during Phase 1 improvements.

---

## Vision

Each resort detail page gains a **"How people experienced it"** section showing authentic visitor experiences. Over time, reviews can be **summarized into themes/genres** (e.g. "great for powder hunters", "crowded during CNY") to complement editorial highlights/lowlights.

---

## Functional requirement (draft FR-8)

| ID | Requirement |
|----|-------------|
| **FR-8** | **Resort experience reviews** — logged-in users can submit structured experience feedback per resort; detail page displays aggregated community signal + individual reviews when authenticated |

### Acceptance criteria (high level — refine when epic is scheduled)

**Given** I am logged in and viewed a resort detail page  
**When** I submit an experience review (rating + short text + optional tags)  
**Then** my review is stored against my user ID and `resort_slug`

**And** the review appears in the resort's **Experience** section with my display name (or anonymous option — TBD)

**Given** any visitor on a resort detail page  
**When** the Experience section renders  
**Then** it shows **aggregate signals** (avg rating, review count, optional AI-generated theme summary when N≥threshold)

**And** full individual reviews are visible only when **logged in** (PO requirement: "also show how exactly review it" for authenticated users)

**And** moderation + reporting flow exists before public launch of UGC (spam, PII, off-topic)

---

## Dependencies

| Dependency | Phase | Notes |
|------------|-------|-------|
| User accounts / auth | Phase 2 | Supabase or equivalent — PRD §9 out of scope Phase 1 |
| Database for reviews | Phase 2+ | Not markdown — runtime UGC |
| AI summarization of reviews | Phase 3 | Optional genre/themes; tie to FR-6 cost guardrails if same provider |
| Moderation tooling | Phase 3 | Human review queue or automated filters |

---

## Analytics (future — tracking-plan §3)

| Event | Trigger | Key properties |
|-------|---------|----------------|
| `review_section_viewed` | Experience section enters viewport | `resort_slug` |
| `review_submitted` | User posts review | `resort_slug`, `rating`, `has_text` |
| `review_summary_expanded` | User expands AI/theme summary | `resort_slug` |

---

## Implementation order (when Phase 3 community starts)

1. **Accounts** — prerequisite from Phase 2
2. **Review data model + API** — create/read per resort, rate limits
3. **Detail UI section** — below Getting There or above Practical Tips (placement TBD in UX pass)
4. **Aggregate + theme summary** — batch job or on-demand AI when review count threshold met
5. **Moderation** — admin flag, delete, tos linkage

---

## Phase 1 / Epic 6 exclusions

- No review UI stub on detail pages
- No placeholder "Coming soon" unless PO explicitly requests teaser copy
- Editorial **highlights/lowlights** remain the sole honesty signal until FR-8 ships

---

**Related:** [`phase-3-monetisation-ai.md`](phase-3-monetisation-ai.md) (AI infra may power summaries)  
**PRD:** [`../prd/06-features-ai-chat.md`](../prd/06-features-ai-chat.md) (AI guardrails)
