<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.4: Ski Passport & Badges (FR-11)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.4 · **See also:** **Epic:** [`epic-13-passport-profile.md`](../../../epics/phase2/shards/epic-13-passport-profile.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.4 Ski Passport & Badges (FR-11)

**Description:** Gamified **Hall of Fame** — users manually check in to resorts they've visited, optionally attach a photo, and earn badges. Warm, playful tone. Public profile shows badge grid. Manual check-in only (no evidence required). Future Slope partnership noted but not in scope.

Realizes UJ-3.

#### FR-11.1: Manual check-in

**User** can tap **I've visited** on a resort detail page once per resort.

**Consequences:**
- Check-in recorded with `resort_slug`, `visited_at` (user-editable date optional `[ASSUMPTION: default today]`), optional note.
- User can undo check-in within 24 hours.
- One check-in per user per resort.

#### FR-11.2: Optional passport photo

**User** can attach up to 1 photo per check-in (10 MB max) displayed on passport entry.

**Consequences:**
- Photo stored in object storage (Supabase Storage); not required for check-in.
- Photo moderated same as review photos (report + admin hide).

#### FR-11.3: Badge awards

**System** evaluates badge rules on each check-in and awards badges automatically.

**Consequences:**
- Badge unlock triggers in-app toast + `badge_earned` analytics event.
- Badge definitions in addendum § Badge Catalog.
- Badges visible on user public profile and next to review author name.

#### FR-11.4: Passport view

**User** can view their Ski Passport: grid of visited resorts, badge shelf, optional stats (total resorts, regions).

**Consequences:**
- Own passport at `/passport` (logged in) or via hamburger menu.
- Public read-only view linked from public profile (FR-17).
- Guest can view others' public passports (encourages aspiration).

**Out of Scope Phase 2:**
- Slope account linking.
- Evidence verification (lift pass OCR, GPS).
- Leaderboards (consider if community stays healthy).

---
