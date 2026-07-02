<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.9: Information Architecture & Navigation (FR-16)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.9 · **See also:** **Epic:** [`epic-10-nav-saved.md`](../../../epics/phase2/shards/epic-10-nav-saved.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.9 Information Architecture & Navigation (FR-16)

**Description:** Phase 2 adds Saved, Plan, and Passport without crowding mobile or feeling like a "fake app" on desktop web. **Adaptive navigation:** bottom tab bar on mobile, horizontal top nav on desktop; secondary destinations in hamburger menu.

**Decision:** Do **not** remove bottom nav on mobile — Powri remains mobile-first (375–430px primary) and thumb-zone navigation still wins on mobile web. Do **not** put 5+ items in bottom nav — use **4 primary tabs + hamburger overflow**. On desktop (≥768px), **hide bottom nav** and show top nav instead — this addresses web-only rollout feeling.

#### FR-16.1: Mobile navigation (<768px)

**Guest or User** sees fixed bottom nav with **4 items:** Home · Explore · Saved · Plan.

**Consequences:**
- Bottom nav always visible; never hide on scroll (Phase 1 behaviour preserved).
- **Passport**, **Info**, **Account / Sign in** live in **hamburger menu** (top bar — extend existing `AppMenuSheet`).
- Plan tab: logged-out users see empty state + sign-in CTA; no login wall on tab itself.
- Active state styling matches Phase 1 Theme A.

#### FR-16.2: Desktop navigation (≥768px)

**Guest or User** sees **horizontal top nav** instead of bottom nav: Home · Explore · Saved · Plan · Passport · Info · Sign in / Avatar.

**Consequences:**
- Bottom nav hidden at **`md` breakpoint (768px)** and above — **OQ-3 resolved**.
- Top nav in header bar; sticky on scroll.
- Same routes as mobile — no desktop-only dead ends.

#### FR-16.3: Hamburger menu (mobile overflow)

**Guest or User** opens hamburger from top bar to reach Passport, Info, Account, Sign in/out.

**Consequences:**
- Extend `AppMenuSheet` nav items — do not duplicate bottom-nav destinations in hamburger (Home/Explore/Saved/Plan stay in bottom bar only).
- Passport shows badge count chip if user has ≥1 badge.
- `nav_menu_opened` event retained; add `nav_destination` property on nav taps.

**Rationale (vs alternatives considered):**

| Option | Verdict |
|--------|---------|
| Hamburger-only (no bottom nav) | Rejected — extra tap for core flows on mobile; hurts discovery |
| 5-item bottom nav (add Passport) | Rejected — crowded at 375px; icon labels truncate |
| Bottom nav on desktop too | Rejected — feels wrong on web; users expect top nav |
| **4-tab mobile + top desktop + hamburger overflow** | **Selected** — scales Phase 2 without UX debt |

---
