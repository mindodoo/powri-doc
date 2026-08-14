---
title: Powri Epic UX — Post–v1 polish
created: 2026-08-14
updated: 2026-08-14
status: final
phase: 2
epic_id: epic-ux
version_namespace: 2.12.2
builds_on: Phase 2 feature epics 9–12 (auth, nav, map, reviews)
---

# PRD: Epic UX — Post–v1 polish

**Audience:** Product, engineering, UX.  
**Stories:** [`implementation-artifacts/UX/`](../../../implementation-artifacts/UX/) (`ux-1`–`ux-13`). No `epics/epic-ux.md` listing.  
**Does not replace:** [`prd-phase2.md`](prd-phase2.md) (feature scope). This file is a **standalone** UX amendment. Do not shard it into `prds/phase2/shards/`.

---

## 0. Document purpose

Phase 1–2 shipped a working trip planner (discovery, detail, accounts, maps, reviews). This PRD captures **UX-only** improvements after that first version: scanability on resort detail, navigation chrome, brand assets, list layout, and app-version display.

**No new product features.** Same content model and same capabilities; clearer presentation and fewer chrome bugs.

---

## 1. Purpose and success

| Goal | Outcome |
|------|---------|
| Resort detail | Stats and verdicts scan as infographics and punchlines, not a wall of text |
| Information flow | Mountain → trail map → nearby places |
| Chrome | Consistent logo, search, and account access; no duplicate or hidden controls |
| Trust | Sign-out and resort Back never land on a dead or wrong screen |
| Version | About / `NEXT_PUBLIC_APP_VERSION` matches the last **shipped** story without a manual env edit |

**Counter-metric:** Do not drop facts that already exist in resort content. Shorter copy and denser layout; same data (except **vertical drop**, which is removed from the on-page stats grid — see UX-2).

---

## 2. Timing and naming

| Item | Value |
|------|--------|
| Timing | After Epic 12 (community reviews) on production |
| Planning name | **epic-ux** (not a running feature-epic number) |
| Requirement IDs | **UX-1 … UX-13** |
| App version | Four-part numeric: **`2.12.2.{n}`** (see §5) |

---

## 3. Functional requirements

| ID | Requirement |
|----|-------------|
| **UX-1** | Resort detail: **one Highlights box** and **one Lowlights box**. **Same visual language for both** (same colors — not green vs red). Each existing highlight/lowlight bullet is **rewritten by engineering** into a short, concise punchline. Do not CSS-truncate long sentences. |
| **UX-2** | Resort detail: show these **six** values in a **3×2 infographic grid** (3 columns × 2 rows), **no cell borders**: top elevation, base elevation, number of runs, longest run, steepest_gradient_deg, lifts. **Do not show vertical drop.** Separate **3×1** row for `terrain_split`: labels **Beginner**, **Intermediate**, **Advanced**, each with the **percentage underneath** (e.g. `24%`). Desktop: both grids **spread proportionally** across the content width. |
| **UX-3** | Resort detail: move **Around the resort** (map + place list) to **immediately after Trailmap**. Intended flow: resort/mountain information → trail map → nearby places. Reorder only. |
| **UX-4** | Report-review modal: keep the current mobile treatment; **increase width on desktop** so it reads as a modal, not a small popover. |
| **UX-5** | After **sign-out**, **do not flash a sign-in page**. Redirect **straight to Home**. That flash is not a usable sign-in surface. |
| **UX-6** | Resort detail **Back** always goes to **Explore** (`/explore`), **not** browser history. After submitting a review, Back must not reopen the review flow. |
| **UX-7** | **Account** page: **remove the Back button**. Keep the hamburger on **mobile** so the user can reach any section. |
| **UX-8** | Add a **favicon** on production. **Product owner provides the asset.** Engineering wires it into app metadata. Story is blocked until the file is in the repo (e.g. `web/public/`). No placeholder artwork. |
| **UX-9** | **Mobile only:** remove the always-visible **search box** from **Account** and **Explore** top bars. On **Explore (mobile)**, put a **search icon on the right**, same as Home; tap opens the **existing Home search panel** (reuse, do not rebuild). **Desktop unchanged:** search icon already sits in the top bar at all times. |
| **UX-10** | **Powri logo** (SVG **provided by product owner**). **Mobile:** centered in the top bar on Saved, Plan, Passport, About, Account. **Home (mobile):** centered between hamburger and search icon. **Desktop:** logo **replaces the “POWRI” text**; combined with UX-13 there is **no hamburger**. Story blocked until the SVG is in the repo. No placeholder mark. |
| **UX-11** | Resort lists on **Home, Explore, Saved**: **responsive multi-column grid from tablet up** (e.g. 3 columns tablet, 4 columns desktop). **Mobile stays a single column.** Card internals unchanged except layout. |
| **UX-12** | App version updates from **shipped story status**, not a manual edit of `NEXT_PUBLIC_APP_VERSION` / `appMeta.ts` each story. See §5. |
| **UX-13** | **Desktop:** **remove the hamburger**. Today that menu only contains Passport and Saved. Move **Account, Saved, and Passport** to a **dropdown on the profile picture**. **Signed out:** keep **Sign in** in the top bar; **hide Saved and Passport** until signed in. **Mobile:** hamburger stays. |

### 3.1 Desktop chrome (UX-10 + UX-13 + UX-9)

Confirmed layout:

**Signed in:** `[Logo] ……………… [search icon] [avatar → Account / Saved / Passport]`

**Signed out:** `[Logo] ……………… [search icon] [Sign in]`

No hamburger. No “POWRI” text. Search icon remains always visible (existing desktop behaviour).

### 3.2 Mobile chrome (UX-7, UX-9, UX-10)

- **Home:** hamburger left · logo center · search icon right (opens existing search panel).
- **Explore:** hamburger left · (page title as today) · search icon right (same panel as Home). No inline search field.
- **Account:** hamburger left · logo center. **No Back button. No search box.**
- **Saved, Plan, Passport, About:** hamburger left · logo center.

---

## 4. Brand assets (owner-provided)

| Asset | Format | Requirement | Blocked until |
|-------|--------|-------------|----------------|
| Favicon | file provided by PO | UX-8 | File committed under `web/public/` (or equivalent Next.js metadata path) |
| Wordmark / mark | SVG provided by PO | UX-10 | SVG committed for nav use |

Recorded here as **information**: artwork is not invented in this epic.

---

## 5. App version (UX-12)

| Piece | Rule |
|-------|------|
| **Docs / epic name** | `epic-ux` |
| **Display string** | Numeric only: **`2.12.2.{n}`** where `{n}` is the UX requirement number (UX-1 → `2.12.2.1`, … UX-13 → `2.12.2.13`) |
| **Meaning** | Phase **2** · post–Epic **12** UX wave **`.2`** · story **n** |
| **Source of truth** | Last story in `sprint-status.yaml` with status **`done`** (or equivalent last-merged-to-`main`). **Not** in-progress work |
| **Mechanism** | **Build-time** derivation into `DEFAULT_APP_VERSION` / `NEXT_PUBLIC_APP_VERSION`. Env override allowed for emergencies only |
| **Stop** | Hand-editing `web/src/lib/site/appMeta.ts` or `.env.local` every story |

**Why not bump from the story in progress:** parallel branches and preview deploys would fight over the number; users would see unreleased versions.

**Relation to `2.12.4`:** Community reviews shipped as three-part `2.12.4`. This wave uses a **four-part** string so it cannot be confused with Epic 12 stories `12.1`–`12.4`. Compare versions by scheme (three-part feature vs `2.12.2.*` UX wave), not by naive semver against `2.12.4`.

---

## 6. Non-goals

- New product features (reviews, trips, AI, Japanese locale, new resort schema fields).
- Redesigning card internals, quiz, or trail-map interaction.
- Inventing favicon or logo artwork.
- Changing the **meaning** of highlights/lowlights — only presentation and punchline length.
- Showing **vertical drop** on the stats grid (explicitly dropped to fit 3×2).
- Changing **desktop** search (already icon-in-bar).
- Removing the **mobile** hamburger.

---

## 7. Analytics

No new events required for this epic unless implementation introduces a new control that is not already covered (`nav_menu_opened`, `search_focused`, sign-in, report submit).

If the avatar dropdown is a new open action, add **one** event when the epic listing is written; until then reuse existing nav/auth events.

Update [`docs/analytics/tracking-plan.md`](../../../docs/analytics/tracking-plan.md) in the same PR **only if** a new event ships.

---

## 8. Decisions (locked)

| # | Decision |
|---|----------|
| D1 | Vertical drop omitted from the stats grid so six values fill 3×2. |
| D2 | Engineering rewrites highlight/lowlight punchlines from current bullets. |
| D3 | Terrain 3×1: category label + percentage below (no bars required). |
| D4 | Desktop signed-out: Sign in remains; Saved/Passport hidden until signed in. |
| D5 | Version string numeric `2.12.2.{n}`; planning name stays `epic-ux`. |
| D6 | Desktop top bar: logo · search icon · avatar or Sign in. No hamburger, no “POWRI” text. |
| D7 | UX-9 search-field removal + Explore search-icon pattern is **mobile-only**. |

---

## 9. Downstream

| Next | Status |
|------|--------|
| Story files (`UX/ux-*.md`) | **ready-for-dev** — implementation not started |
| Epic monolith (`epics/…/epic-ux.md`) | Not used — PRD + `UX/` stories are source |
| UX spec / design.md updates | Optional; this PRD is sufficient for chrome and layout |
| Architecture | Not required (presentation and nav only; version script is app-side) |
