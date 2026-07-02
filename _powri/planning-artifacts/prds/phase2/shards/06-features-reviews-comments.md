<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.3: Reviews & Comments (FR-10)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.3 · **See also:** **Epic:** [`epic-12-community-reviews.md`](../../../epics/phase2/shards/epic-12-community-reviews.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.3 Reviews & Comments (FR-10)

**Description:** Logged-in users submit **public reviews** (rating + text + optional photos) and **comments** on resort detail pages. Reviews show author profile (name + avatar + badges). No anonymous mode — encourages thoughtful, accountable contributions. Moderation: users report; admins hide/delete via API (admin UI Phase 3).

**Experience section layout (decided):** Single **"How people experienced it"** section on resort detail (addendum §M):
1. Aggregate rating + review count (SSR-friendly)
2. Review list (rating, text, photos, author profile link)
3. **Flat comment thread** below reviews — no nested replies in Phase 2
4. CTAs: "Write a review" / "Add a comment" (sign-in gated)

Realizes UJ-2, UJ-5.

#### FR-10.1: Submit review

**User** can submit one review per resort (editable after submit); includes 1–5 star rating, text (max 2,000 chars), up to **5 photos** (10 MB each).

**Consequences:**
- Review appears in **Experience** section on resort detail.
- Aggregate display: average rating + review count visible to **Guest and User**.
- Full review list visible to all (public reviews with identity).
- `review_submitted` analytics event fires on success.

#### FR-10.2: Comment on resort

**User** can post comments (max 500 chars) in a **flat comment thread** below the review list on resort detail — **no nested replies, no threading** in Phase 2.

**Consequences:**
- Comments show author profile + timestamp.
- Rate limit: 10 comments/hour/user.
- Comments are visually distinct from reviews (no star rating, lighter card style).

#### FR-10.3: Report review or comment

**User** (logged in only) can report a review or comment with reason enum (`spam`, `offensive`, `off_topic`, `other`).

**Consequences:**
- Guest tapping Report sees sign-in sheet with `trigger: report`.
- Report creates moderation queue record; reporter notified "Thank you — we'll review."
- Duplicate report same item by same user blocked.

#### FR-10.4: Admin moderation (API-level)

**Admin** can hide or delete reviews and comments via authenticated admin API or direct DB (no admin UI Phase 2).

**Consequences:**
- Hidden content not rendered for any user.
- Delete is soft-delete with audit log (admin_id, timestamp, action).
- Minimum requirements met: Guest/User report; Admin hide/delete.

**Out of Scope Phase 2:**
- AI-generated review theme summaries (Phase 3).
- Review voting / helpful counts (consider Phase 3).
- Anonymous reviews.

---
