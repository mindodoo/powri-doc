<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.10: Public Profile (FR-17)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.10 · **See also:** **Epic:** [`epic-13-passport-profile.md`](../../../epics/phase2/shards/epic-13-passport-profile.md) · [`INDEX.md`](../../../INDEX.md)

---

### 5.10 Public Profile (FR-17)

**Description:** Each user has a **public profile page** showcasing their community presence — badges, passport grid, and published reviews. Supports the "warm, accountable identity" goal for reviews and passport sharing.

Realizes UJ-3 (resolution — share passport URL).

#### FR-17.1: Public profile page

**Guest or User** can view `/u/:username` showing display name, avatar, badge shelf, visited-resort grid, and list of public reviews (resort name + rating + excerpt).

**Consequences:**
- Username slug derived from display name (unique, URL-safe; collision suffix `-2`, etc.).
- No email or private data exposed.
- Owner can edit profile from `/account` (logged in).
- `noindex, follow` meta on profile pages Phase 2 (FR-14.3).
- Links from review author name → public profile.

#### FR-17.2: Account settings

**User** can access `/account` to edit display name, bio, profile photo (upload); export data; request deletion; sign out.

**Consequences:**
- Linked from hamburger / avatar menu.
- Photo upload auto-saves; profile photo preview and reload reflect the stored `avatar_url`; initials shown when no photo.
- Avatar validation errors (file type, size) display inline under the upload control; form field errors display above Save — use semantic `--color-error` / `.text-error` (Theme A token).
- Deletion and export per FR-8.4.

---
