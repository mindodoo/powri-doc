<!-- generated from prds/phase2/prd-phase2.md 2026-07-02 — do not edit -->

# PRD §5.5: Trip Planning Skeleton (FR-12)

**Source:** [`../prd-phase2.md`](../prd-phase2.md) §5.5 · **See also:** **Epic:** [`epic-14-trip-planning.md`](../../../epics/phase2/shards/epic-14-trip-planning.md) · **Addendum:** trip templates + schema · [`INDEX.md`](../../../INDEX.md)

---

### 5.5 Trip Planning Skeleton (FR-12)

**Description:** Logged-in users create up to **5 trips** each. Trips use a **template itinerary** (placeholder activities) — not real accommodations or restaurants. Users can edit titles, notes, reorder activities, and delete trips to free quota. Schema designed for Phase 3 AI population. AI "Fill trip" button present as disabled stub with copy explaining Phase 3.

Realizes UJ-4.

#### FR-12.1: Create trip (quota 5)

**User** can create a new trip with: title, primary `resort_slug`, `start_date`, `end_date`, status (`draft` | `planned` | `completed`).

**Consequences:**
- Max **5 trips per user** at any time; user must delete a trip to create a 6th.
- Deletion is permanent (confirm dialog).
- `trip_created` analytics event.

#### FR-12.2: Template itinerary generation

**System** generates default `itinerary_days` from date range and resort template (see addendum § Trip Templates).

**Consequences:**
- Each day has ordered activities with types: `airport`, `transport`, `accommodation`, `ski`, `restaurant`, `attraction`, `custom`.
- Placeholders have generic titles ("Airport arrival", "Check in to accommodation", "Dinner nearby") — no real POI IDs.
- User can add/remove/reorder activities and edit title + notes.

#### FR-12.3: Trip list & detail views

**User** can view all trips in Plan tab/surface and open day-by-day itinerary view.

**Consequences:**
- Mobile-first vertical timeline UI.
- Empty state CTA: "Plan your first trip" with resort picker.
- Trip data persisted in PostgreSQL (not localStorage).

#### FR-12.4: AI handoff readiness

**System** exposes trip schema and `ai_ready: true` flag on trips with resort + dates set; stores `entry_point` context for Phase 3.

**Consequences:**
- Documented JSON schema in addendum § Trip Object Schema.
- Disabled **"Fill with AI"** CTA on trip detail — visible, explains "Coming in Phase 3".
- Entry points on quiz results page and resort detail: **"Plan this trip with AI"** stub (same disabled state).
- Quiz results **not** stored on user profile Phase 2; Phase 3 AI receives quiz answers only from in-session handoff or explicit re-take `[per PO decision]`.

**Out of Scope Phase 2:**
- Real hotel/restaurant/activity data or booking links in itinerary.
- Multi-resort trips (single primary resort Phase 2 `[ASSUMPTION]`).
- Sharing/editing trips with collaborators.
- AI population of itinerary.

---
