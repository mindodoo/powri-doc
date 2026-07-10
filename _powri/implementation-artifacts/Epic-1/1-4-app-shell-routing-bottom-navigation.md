# Story 1.4: App Shell, Routing & Bottom Navigation

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **user**,
I want persistent bottom navigation between Home, Explore, and Info,
So that I can move around the app without losing context.

## Acceptance Criteria

1. **Given** the app is open on any main tab  
   **When** I view the screen on a 375–430px viewport  
   **Then** `BottomNav` shows Home · Explore · Info with active-state indicator (UX-DR3, NFR-1)

2. **Given** any scrollable main tab page  
   **When** the user scrolls  
   **Then** the nav bar remains visible and does not hide on scroll (§12.5)

3. **Given** the app router  
   **When** routes are inspected  
   **Then** routes exist for `/`, `/explore`, `/info`, and `/resorts/[slug]` (placeholder pages acceptable until Epic 2–3)

4. **Given** Theme A tokens from Story 1.2  
   **When** any shell page renders  
   **Then** mobile-first layout uses warm off-white `--color-bg` page background

## Tasks / Subtasks

- [x] **BottomNav component** (AC: 1, 2)
  - [x] Create `web/src/components/layout/BottomNav.tsx` (client component — `usePathname` for active tab)
  - [x] Three tabs: Home (`/`), Explore (`/explore`), Info (`/info`)
  - [x] Lucide line icons 24px; DM Sans 11px labels; 56px height; `--color-nav-bg` + top border `--color-divider`
  - [x] Active tab: `--color-accent` label + dot indicator (per mockup `key-home.html`)
  - [x] Tap targets ≥44px (NFR-10); include `safe-area-inset-bottom` padding

- [x] **App shell** (AC: 2, 4)
  - [x] Create `web/src/components/layout/AppShell.tsx` — wraps children with `bg-bg`, main scroll area, fixed BottomNav
  - [x] Wire `AppShell` in `web/src/app/layout.tsx`
  - [x] Main content area has bottom padding so content is not hidden behind nav

- [x] **Routes & placeholders** (AC: 3, 4)
  - [x] Refactor `web/src/app/page.tsx` — Home placeholder (keep content loader status or simplify)
  - [x] Create `web/src/app/explore/page.tsx` — Explore placeholder
  - [x] Create `web/src/app/info/page.tsx` — Info placeholder
  - [x] Create `web/src/app/resorts/[slug]/page.tsx` — placeholder with resort name from loader
  - [x] Add `getResortBySlugFromFiles(slug)` (all files, incl. draft) + `getAllResortSlugs()` to content module for static params

- [x] **Tailwind token** (AC: 1)
  - [x] Expose `--color-nav-bg` in `globals.css` `@theme inline` if not already (for `bg-nav-bg`)

- [x] **Validate** (AC: 1–4)
  - [x] `npm run build` passes (20 resort static routes from draft files)
  - [x] `npm run lint` passes
  - [x] Manual: nav visible on all four route types; active state correct on tabs

- [x] **Out of scope — do NOT implement**
  - Hero carousel, resort cards, filters (Epic 2)
  - Full Info tab content (Story 5.1)
  - Full resort detail UI (Epic 3)
  - Analytics (Story 1.5)
  - Plan tab (Phase 3)

## Dev Notes

### BottomNav spec (DESIGN.md / mockups)

| Property | Value |
|----------|-------|
| Height | 56px (`h-14`) |
| Background | `--color-nav-bg` (#FFFFFF) |
| Border | `1px solid --color-divider` top |
| Labels | DM Sans 11px / 500 |
| Icons | Lucide, 24px stroke |
| Active | `--color-accent` text + 4px dot below label |
| Inactive | `--color-text-secondary` |

**Active path matching:**
- `/` → Home (exact match only — not `/resorts/...`)
- `/explore` → Explore
- `/info` → Info
- `/resorts/[slug]` → no tab active (detail is off-tab)

### App shell layout pattern

```tsx
<div className="flex min-h-full flex-col bg-bg">
  <main className="flex-1 pb-[calc(3.5rem+env(safe-area-inset-bottom))]">
    {children}
  </main>
  <BottomNav /> {/* fixed bottom-0 */}
</div>
```

Nav is `fixed` — never hides on scroll. Main content scrolls independently.

### Resort route placeholder

Use `loadAllResortFiles()` / `getAllResortSlugs()` for `generateStaticParams` — includes draft files so all 20 slugs pre-render at build time. Display `nameEn` from file even when `content_status: draft`. Do **not** use `getAllResorts()` (published-only) for placeholder name lookup.

Add to content module:

```typescript
export function getResortBySlugFromFiles(slug: string): Resort | undefined;
export function getAllResortSlugs(): string[];
```

### Dependencies

- `lucide-react` — icons per DESIGN.md §12.9 / UX spine
- No other new packages

### Previous story intelligence

- Story 1.3: content loader at `@/lib/content`; home page already shows loader status
- All resorts draft — resort placeholders show names from files, not published filter
- Theme tokens ready — use `bg-bg`, `text-text-primary`, `border-divider`, etc.

### Testing requirements

- **Automated:** `npm run build`, `npm run lint`
- **Manual:** Navigate all tabs; verify active dot; scroll long page — nav stays fixed
- **Manual:** Visit `/resorts/niseko-united` — placeholder shows "Niseko United"

### References

- [Source: epics.md § Story 1.4](../planning-artifacts/epics/phase1/epics-phase1.md)
- [Source: DESIGN.md § BottomNav](../planning-artifacts/ux-designs/ux-phase1/design.md)
- [Source: EXPERIENCE.md § IA](../planning-artifacts/ux-designs/ux-phase1/experience.md)
- [Source: mockups/key-home.html](../planning-artifacts/ux-designs/ux-phase1/mockups/key-home.html)
- [Source: Story 1.3](./1-3-markdown-content-loader-resort-types.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — 27 static pages (/, /explore, /info, 20× /resorts/[slug], + health)
- `npm run lint` — pass

### Completion Notes List

- Added `BottomNav` (Lucide Home/Compass/Info, accent dot active state, fixed 56px + safe area)
- Added `AppShell` wired in root layout — warm `bg-bg`, nav never hides on scroll
- Placeholder pages for Home, Explore, Info; resort detail placeholder with `generateStaticParams` for all 20 slugs
- Extended content loader with `getResortBySlugFromFiles` + `getAllResortSlugs` for draft-aware routing

### File List

- `web/package.json` (modified — lucide-react)
- `web/package-lock.json` (modified)
- `web/src/app/globals.css` (modified — bg-nav-bg token)
- `web/src/app/layout.tsx` (modified — AppShell)
- `web/src/app/page.tsx` (modified)
- `web/src/app/explore/page.tsx` (new)
- `web/src/app/info/page.tsx` (new)
- `web/src/app/resorts/[slug]/page.tsx` (new)
- `web/src/components/layout/AppShell.tsx` (new)
- `web/src/components/layout/BottomNav.tsx` (new)
- `web/src/lib/content/loadResorts.ts` (modified)
- `web/src/lib/content/index.ts` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve

### Summary

All four ACs satisfied. Fixed BottomNav with Lucide icons, accent active state, 56px height, safe-area padding, and routes for `/`, `/explore`, `/info`, plus 20 SSG `/resorts/[slug]` placeholders.

### Review Findings

- [x] [Review][Defer] Consent banner will stack above BottomNav in Story 1.5 — coordinate z-index
- [x] [Review][Defer] `min-h-11` (44px) meets NFR-10; full-width flex tab area could be taller on some devices — acceptable
