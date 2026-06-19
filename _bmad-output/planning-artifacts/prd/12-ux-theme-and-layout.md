# PRD §12.0–12.4: UX Theme & Layout

**Source:** [`../prd.md`](../prd.md) §12.0–12.4  
**Implementation authority:** [`../ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md`](../ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md) — **wins on conflict**

---

## Design language (§12.1)

Premium travel magazine: warm off-white, photography as hero, editorial typography, calm motion.

**References:** Kinfolk (breathing room), Hokuoh (image→text stack), Monocle (section headers), Airbnb (filters, cards, bottom nav).

---

## Theme A — Default (active V1)

| Token | Value |
|-------|-------|
| `--color-bg` | Warm off-white (#FAF8F5 area) |
| `--color-text` | Deep charcoal |
| `--color-accent` | Icy blue |
| `--color-nav-bg` | White with border |

Themes B/C/D deferred until i18n phases.

**Full token table:** DESIGN.md or [`../prd.md`](../prd.md) §12.2 Theme A

---

## Typography — Default (§12.3)

| Role | Font |
|------|------|
| Display / headings | Cormorant Garamond |
| Body / UI | DM Sans |

Load from Google Fonts. Bottom nav labels: DM Sans 11px.

---

## Layout (§12.4)

- Viewport: **375–430px** primary
- 8px spacing grid
- Bottom nav: **Home · Explore · Info** — always visible, never hide on scroll
- Card radius, button radius per Theme A table

**§5 UI/UX section is superseded by §12** — use §12 + DESIGN.md for implementation.
