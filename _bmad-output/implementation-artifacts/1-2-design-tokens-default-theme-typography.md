# Story 1.2: Design Tokens & Default Theme Typography

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **user**,
I want the app to look like a premium travel magazine with consistent colours and type,
So that every screen feels cohesive and trustworthy.

## Acceptance Criteria

1. **Given** Theme A (Default) tokens in PRD §12.2–12.3  
   **When** any page renders  
   **Then** CSS custom properties implement all Default Theme colour tokens (UX-DR1)

2. **Given** the typography spec in DESIGN.md  
   **When** fonts load  
   **Then** Cormorant Garamond and DM Sans load from Google Fonts with specified weights/sizes (UX-DR2)

3. **Given** Theme A spacing and shape tables  
   **When** layout utilities are inspected  
   **Then** spacing follows 8px grid; card, button, and badge radii match Theme A table

4. **Given** typography line-height rules  
   **When** body and display text render  
   **Then** body line-height is 1.6; display headings 1.15

## Tasks / Subtasks

- [x] **Create tokens.css** (AC: 1, 3, 4)
  - [x] Create `web/src/styles/tokens.css` with `:root[data-theme="default"]` block
  - [x] Define all colour tokens: `--color-bg` through `--color-nav-bg`, plus `--color-on-accent`, `--color-overlay-scrim`, `--color-hero-gradient`
  - [x] Define spacing tokens: `--space-1`…`--space-5`, `--space-screen-x`, `--space-card-gap`, `--space-section-gap`, `--space-card-padding`
  - [x] Define radius tokens: `--radius-card`, `--radius-button`, `--radius-badge`, `--radius-pill`, `--radius-image`
  - [x] Define typography size/line-height/letter-spacing tokens per DESIGN.md

- [x] **Wire fonts in layout** (AC: 2, 4)
  - [x] Replace Geist fonts with `Cormorant_Garamond` (500, 600, italic 400) and `DM_Sans` (400, 500, 600) via `next/font/google`
  - [x] Set `<html data-theme="default" lang="en">` with font CSS variables
  - [x] Map `--font-display` and `--font-body` in tokens.css to next/font variables

- [x] **Update globals.css** (AC: 1, 3, 4)
  - [x] Import `../styles/tokens.css` before Tailwind
  - [x] Remove create-next-app dark-mode `:root` overrides and Geist font references
  - [x] Extend `@theme inline` to expose token colours, fonts, spacing, radii to Tailwind utilities
  - [x] Set `body` background to `var(--color-bg)`, color to `var(--color-text-primary)`, font to `var(--font-body)`, line-height 1.6

- [x] **Apply tokens to starter page** (AC: 1, 2, 4)
  - [x] Update `web/src/app/page.tsx` — remove zinc/dark/Geist classes; use token-based Tailwind classes (`bg-bg`, `text-text-primary`, etc.)
  - [x] Minimal placeholder content OK — proves tokens render on a real page

- [x] **Validate** (AC: 1–4)
  - [x] `npm run build` passes
  - [x] `npm run lint` passes
  - [x] Visual sanity: warm off-white page bg (`#F7F5F2`), not pure white canvas

- [x] **Out of scope — do NOT implement**
  - BottomNav, components, content loader (Stories 1.3–1.4)
  - Theme B/C/D switching UI
  - Lucide icons
  - Full Home/Resort UI

## Dev Notes

### Critical constraints

| Rule | Detail |
|------|--------|
| **Token file location** | `web/src/styles/tokens.css` per architecture.md — NOT inline-only in globals.css |
| **Theme attribute** | Phase 1 ships Theme A only; use `data-theme="default"` on `<html>` for future swap |
| **Import alias** | `@/*` → `src/*` only |
| **No new dependencies** | Fonts via `next/font/google` only — no `@fontsource` packages |
| **Tailwind v4** | Follow generated template `@import "tailwindcss"` + `@theme inline` pattern — do not downgrade to v3 config |

### Theme A colour tokens (exact values)

From PRD §12.2 / DESIGN.md:

| Token | Value |
|-------|-------|
| `--color-bg` | `#F7F5F2` |
| `--color-surface` | `#FFFFFF` |
| `--color-text-primary` | `#1C1A17` |
| `--color-text-secondary` | `#6E6A63` |
| `--color-accent` | `#2D6BE4` |
| `--color-accent-warm` | `#C4873A` |
| `--color-divider` | `#E8E5E0` |
| `--color-highlight-bg` | `#EEF3FF` |
| `--color-lowlight-bg` | `#FFF4E3` |
| `--color-nav-bg` | `#FFFFFF` |
| `--color-on-accent` | `#FFFFFF` |
| `--color-overlay-scrim` | `rgba(0,0,0,0.3)` |
| `--color-hero-gradient` | `linear-gradient(to top, rgba(0,0,0,0.55) 0%, transparent 60%)` |

PRD structure example:

```css
:root[data-theme="default"] {
  --color-bg: #F7F5F2;
  /* ... */
}
```

### Typography tokens

Google Fonts URL equivalent via next/font:
- `Cormorant_Garamond`: weights 500, 600; italic 400
- `DM_Sans`: weights 400, 500, 600

| Role | Font | Size | Weight | Line-height |
|------|------|------|--------|-------------|
| Display | Cormorant Garamond | 40px | 600 | 1.15 |
| Display italic | Cormorant Garamond | 24px | 400 italic | 1.25 |
| Section heading | Cormorant Garamond | 26px | 500 | 1.2 |
| Card title | DM Sans | 18px | 600 | 1.3 |
| Body | DM Sans | 15px | 400 | **1.6** |
| Caption | DM Sans | 12px | 400 | 1.4 |
| Badge | DM Sans | 11px | 500 | 1.2, letter-spacing 0.08em |
| Nav label | DM Sans | 11px | 500 | 1.2 |

Expose as CSS custom properties (e.g. `--font-size-body`, `--line-height-body`, `--line-height-display`).

### Spacing & radii (PRD §12.4)

| Token | Value |
|-------|-------|
| `--space-1` | 8px |
| `--space-2` | 16px |
| `--space-3` | 24px |
| `--space-4` | 32px |
| `--space-5` | 48px |
| `--space-screen-x` | 20px |
| `--space-card-gap` | 20px |
| `--space-section-gap` | 48px |
| `--space-card-padding` | 16px |
| `--radius-card` | 14px |
| `--radius-button` | 10px |
| `--radius-badge` | 99px |
| `--radius-pill` | 99px |
| `--radius-image` | 0px |

### Tailwind v4 @theme mapping pattern

In `globals.css`, map tokens to Tailwind theme keys so utilities like `bg-bg`, `text-text-secondary`, `rounded-card` work:

```css
@import "../styles/tokens.css";
@import "tailwindcss";

@theme inline {
  --color-bg: var(--color-bg);
  --color-surface: var(--color-surface);
  /* map all colour tokens */
  --font-sans: var(--font-body);
  --font-serif: var(--font-display);
  --radius-card: var(--radius-card);
  /* etc. */
}
```

Use semantic Tailwind color names matching token names (e.g. `--color-text-primary` → utility `text-text-primary`).

### Font wiring pattern (layout.tsx)

```typescript
import { Cormorant_Garamond, DM_Sans } from 'next/font/google';

const cormorant = Cormorant_Garamond({
  variable: '--font-cormorant',
  subsets: ['latin'],
  weight: ['500', '600'],
  style: ['normal', 'italic'],
});

const dmSans = DM_Sans({
  variable: '--font-dm-sans',
  subsets: ['latin'],
  weight: ['400', '500', '600'],
});

// html: data-theme="default" className={`${cormorant.variable} ${dmSans.variable}`}
```

In tokens.css:

```css
--font-display: var(--font-cormorant), Georgia, serif;
--font-body: var(--font-dm-sans), system-ui, sans-serif;
```

### Architecture compliance

- Matches architecture.md: `src/styles/tokens.css` + Tailwind theme extension
- Story 1.3+ components will consume these tokens — establish naming now, do not invent alternate names
- NFR-10 contrast: Theme A values are pre-validated in UX; do not alter hex values

### Previous story intelligence (1.1)

- `web/` scaffold exists; Geist fonts and dark-mode globals are create-next-app defaults — **replace entirely**
- Health route and HSTS unchanged — do not modify `next.config.ts` or `/api/health`
- Nested `web/.git` exists — ignore for this story
- `baseline_commit: NO_VCS` at repo root

### Testing requirements

- **Manual:** `npm run dev` → page background is warm off-white `#F7F5F2`, body text uses DM Sans
- **Manual:** Inspect computed styles — `--color-bg`, `--font-body`, `--line-height-body: 1.6`
- **Automated:** `npm run build` + `npm run lint` must pass
- No unit test files required for token scaffold story

### References

- [Source: epics.md § Story 1.2](../planning-artifacts/epics.md)
- [Source: japan_ski_prd.md §12.2–12.4](../../japan_ski_prd.md)
- [Source: DESIGN.md](../planning-artifacts/ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md)
- [Source: architecture.md § Styling](../planning-artifacts/architecture.md)
- [Source: Story 1.1](./1-1-initialize-next-js-project-deployment-baseline.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — pass (Next.js 16.2.9)
- `npm run lint` — pass

### Completion Notes List

- Created `web/src/styles/tokens.css` with full Theme A `:root[data-theme="default"]` token set
- Replaced Geist with Cormorant Garamond + DM Sans via `next/font/google`; `data-theme="default"` on `<html>`
- Rewired `globals.css` — tokens import, `@theme inline` mapping, body defaults (line-height 1.6)
- Updated starter `page.tsx` to use token utilities; removed zinc/dark mode classes
- Health route and HSTS unchanged from Story 1.1

### File List

- `web/src/styles/tokens.css` (new)
- `web/src/app/globals.css` (modified)
- `web/src/app/layout.tsx` (modified)
- `web/src/app/page.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve  
**Layers:** Blind Hunter, Edge Case Hunter, Acceptance Auditor

### Summary

All four ACs satisfied. Theme A tokens complete in `tokens.css`; fonts wired correctly; spacing/radii match spec; body line-height 1.6 and display 1.15 enforced. Build and lint pass.

### Action Items

- [x] [Review][Defer] `--space-1`…`--space-5` not exposed via `@theme inline` — tokens exist in CSS; Tailwind utilities deferred until component stories
- [x] [Review][Defer] `page.tsx` uses raw `var(--font-size-*)` classes — acceptable for scaffold; typography component utilities in later stories
- [x] [Review][Defer] Placeholder button below 44px tap target — NFR-10 addressed when real Button component lands (Story 1.4+)

### Review Findings

- [x] [Review][Defer] `--space-1`…`--space-5` not in Tailwind theme — defer
- [x] [Review][Defer] Inline CSS var typography on page — defer
- [x] [Review][Defer] Button tap target size — defer to component stories
