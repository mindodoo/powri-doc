# Story UX-10: Powri logo in top nav

Status: ready-for-dev

**Blocked:** Product owner must commit the SVG. **No placeholder mark.** Prefer after UX-9 and UX-13 so chrome slots exist.

Version when done: **2.12.2.10**

<!-- Ultimate context engine analysis completed — comprehensive developer guide created -->

## Story

As a user,  
I want the Powri logo in the top bar,  
so that the product is branded without the “POWRI” text next to the hamburger.

**Source:** [`prds/phase2/prd-epic-ux.md`](../../planning-artifacts/prds/phase2/prd-epic-ux.md) (UX-10, D6, §3.1–3.2).

## Acceptance Criteria

1. Use the **PO-provided SVG** (e.g. `web/src/components/brand/PowriLogo.tsx` wrapping the file, or `next/image` / inline SVG).
2. **Mobile Home:** logo **centered** between hamburger (left) and search icon (right).
3. **Mobile Saved, Plan, Passport, About (`/info`), Account:** logo **centered** in the top bar; hamburger left. About: hamburger + logo (inline search on Info yields to this layout).
4. **Desktop:** logo **replaces** `{t('brand')}` (“POWRI”) text. Combined with UX-13: **no hamburger** on desktop — `[Logo] … centre links … [search] [avatar | Sign in]`.
5. Logo is decorative or has an accessible name; if tappable, Home only (`href="/"`) — do not invent extra destinations.
6. No SVG from the implementer.

## Testing & Definition of Done

- [ ] **Unit:** N/A
- [ ] **Quiz / scoring:** N/A
- [ ] **Analytics:** Optional `nav_destination_tapped` home if logo links home
- [ ] **Content / resorts:** N/A
- [ ] **Around-area labels:** N/A
- [ ] **User-facing flow:** Manual mobile pages in AC3 + Home; desktop logo not text
- [ ] **Manual QA:** PO — asset fidelity

## Tasks / Subtasks

- [ ] Add SVG to repo (`web/src/assets/` or `web/public/`) (AC: 1, 6)
- [ ] Shared `PowriLogo` component (AC: 1)
- [ ] `DiscoveryTopBar` icons layout: three slots hamburger | logo | search (AC: 2)
- [ ] `MobileOverflowNav`: hamburger | logo (AC: 3)
- [ ] `InfoTopBar`: match About chrome (AC: 3)
- [ ] `DesktopTopNav`: logo instead of `nav.brand` text (AC: 4) — hamburger already gone if UX-13 shipped

## Dev Notes

**Today:** Mobile Home/Explore/Info = `DiscoveryTopBar` (`md:hidden`). Other pages = `MobileOverflowNav` (hamburger only). Desktop left cluster = Menu button + serif `{t('brand')}`.

**Coordinate with UX-13:** If this ships first, replace text with logo **beside** hamburger; UX-13 then removes hamburger. If UX-13 ships first, insert logo in the left cluster where brand text was.

**Do not:** Recreate logo in PNG. Put logo in bottom nav.

### Files

| Path | Role |
|------|------|
| PO SVG path | NEW |
| `web/src/components/layout/DiscoveryTopBar.tsx` | Center logo |
| `web/src/components/layout/MobileOverflowNav.tsx` | Center logo |
| `web/src/components/info/InfoTopBar.tsx` | About |
| `web/src/components/layout/DesktopTopNav.tsx` | Replace brand text |
| `web/messages/en.json` `nav.brand` | May remain for a11y |

### References

- PRD UX-10, D6, §3.1–3.2, §4

## Dev Agent Record

### Agent Model Used

Cursor Grok 4.6 (create-story)

### Debug Log References

### Completion Notes List

### File List
