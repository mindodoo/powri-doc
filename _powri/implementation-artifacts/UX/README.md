# Epic UX: Post–v1 polish

UX-only wave after Epic 12. **No new product features.** Planning name `epic-ux`; app version namespace **`2.12.2.{n}`**.

**PRD:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md)  
**Status:** Stories ready-for-dev · implementation **not started**  
**Folder:** `UX/` (not `Epic-<N>/`) so this does not collide with Epic 12 keys like `12-2-1-*`.

## Stories

| # | Version | File | Summary |
|---|---------|------|---------|
| **UX-1** | 2.12.2.1 | [`ux-1-highlight-lowlight-boxes.md`](ux-1-highlight-lowlight-boxes.md) | Highlight/lowlight boxes + punchline copy |
| **UX-2** | 2.12.2.2 | [`ux-2-mountain-stats-terrain-grids.md`](ux-2-mountain-stats-terrain-grids.md) | 3×2 mountain stats + 3×1 terrain split |
| **UX-3** | 2.12.2.3 | [`ux-3-around-resort-after-trailmap.md`](ux-3-around-resort-after-trailmap.md) | Map section immediately after trail map |
| **UX-4** | 2.12.2.4 | [`ux-4-report-review-modal-width.md`](ux-4-report-review-modal-width.md) | Wider report modal on desktop |
| **UX-5** | 2.12.2.5 | [`ux-5-sign-out-no-signin-flash.md`](ux-5-sign-out-no-signin-flash.md) | Sign-out → Home with no sign-in flash |
| **UX-6** | 2.12.2.6 | [`ux-6-resort-back-to-explore.md`](ux-6-resort-back-to-explore.md) | Detail Back always `/explore` |
| **UX-7** | 2.12.2.7 | [`ux-7-account-remove-back.md`](ux-7-account-remove-back.md) | Remove Account overlay Back |
| **UX-8** | 2.12.2.8 | [`ux-8-production-favicon.md`](ux-8-production-favicon.md) | Favicon — **blocked on PO asset** |
| **UX-9** | 2.12.2.9 | [`ux-9-mobile-explore-search-icon.md`](ux-9-mobile-explore-search-icon.md) | Mobile Explore search icon (desktop unchanged) |
| **UX-10** | 2.12.2.10 | [`ux-10-powri-nav-logo.md`](ux-10-powri-nav-logo.md) | Nav logo — **blocked on PO SVG** |
| **UX-11** | 2.12.2.11 | [`ux-11-responsive-resort-list-grid.md`](ux-11-responsive-resort-list-grid.md) | Tablet/desktop resort card grid |
| **UX-12** | 2.12.2.12 | [`ux-12-auto-app-version.md`](ux-12-auto-app-version.md) | Version from last **done** story |
| **UX-13** | 2.12.2.13 | [`ux-13-desktop-avatar-dropdown.md`](ux-13-desktop-avatar-dropdown.md) | Desktop: no hamburger; avatar menu |

## Suggested order

Small chrome first, then detail, then nav that UX-10 depends on, then blocked assets:

`UX-6 → UX-7 → UX-5 → UX-4 → UX-3 → UX-1 → UX-2 → UX-9 → UX-13 → UX-11 → UX-12` then **UX-8 / UX-10** when files exist.

Do **not** start `12-2-*` keys — those are Epic 12 review stories.
