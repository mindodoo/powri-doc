---
baseline_commit: c15a7294e8ab531e30007a00801b8730088f6ef4
---

# Story 10.2.1: Mobile Page Titles Hidden Under Fixed Top Bar

Status: done

**Epic:** 10 · **Parent:** Story [10.2](./10-2-hamburger-overflow-menu.md) (follow-up UX polish, AC 16) · **Shard:** [`epic-10-nav-saved.md`](../planning-artifacts/epics/phase2/shards/epic-10-nav-saved.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.
**Reported:** 2026-07-09 (found on mobile immediately after 10.2's desktop-focused follow-up polish shipped) · **Environment:** Mobile web (<768px), all browsers — confirmed via production build CSS output, not just dev server
**Severity:** P2 — visual regression, page title unreadable/clipped under nav chrome on 6 routes; not a functional blocker but a first-impression quality bug across Explore, Plan, About, Saved, Passport, Account

---

## Story

As a mobile visitor,
I want the page title on Explore, Plan, About, Saved, Passport, and Account to render fully below the fixed mobile top bar,
so that the heading isn't visually hidden/overlapped by the hamburger/search chrome.

---

## Symptom

On mobile (<768px), the `<h1>` rendered by `PageHeader` on **Explore, Plan, About (`/info`), Saved, Passport** (all reported by PO) — and **Account**, which shares the identical mechanism and is very likely affected too — sits directly under the fixed mobile top bar (hamburger, and inline search on Explore/About) instead of below it. Desktop (≥768px) is unaffected.

## Root cause (Confirmed — verified against compiled production CSS, not just source reading)

`web/src/lib/layout/pageInsets.ts` builds the shared top-offset utility classes via **string interpolation of a TS numeric constant into a Tailwind arbitrary-value bracket**:

```7:40:web/src/lib/layout/pageInsets.ts
export const TOP_BAR_HEIGHT_REM = 3.5;

/** Desktop global nav height — matches unified `DesktopTopNav` (`h-14`). */
export const DESKTOP_TOP_NAV_HEIGHT_REM = 3.5;
...
export const TOP_BAR_OFFSET_CLASS =
  `pt-[calc(${TOP_BAR_HEIGHT_REM}rem+env(safe-area-inset-top))]` as const;
...
export const DISCOVERY_PAGE_TOP_OFFSET_CLASS =
  `pt-[calc(${TOP_BAR_HEIGHT_REM}rem+env(safe-area-inset-top))] md:pt-[calc(${DESKTOP_TOP_NAV_HEIGHT_REM}rem+env(safe-area-inset-top))]` as const;
...
export const MAIN_PAGE_TOP_OFFSET_CLASS = DISCOVERY_PAGE_TOP_OFFSET_CLASS;

export const MAIN_PAGE_CONTENT_CLASS =
  `pb-section-gap ${MAIN_PAGE_TOP_OFFSET_CLASS}` as const;
```

Tailwind v4's build scans **raw source text**, not evaluated JS. It only ever sees the literal, un-interpolated fragment `pt-[calc(${TOP_BAR_HEIGHT_REM}rem+env(safe-area-inset-top))]` in the `.ts` file — which is not a valid CSS candidate — and drops it. It never sees the runtime-resolved string `pt-[calc(3.5rem+env(safe-area-inset-top))]` anywhere in static source, so no matching CSS rule is ever generated for it.

**Confirmed by build artifact, not guessed:** running `npm run build` (Turbopack production build) and grepping the emitted CSS chunks (`.next/static/chunks/*.css`) for `pt-[calc(3.5rem` returns **zero matches** — the class renders in the DOM but resolves to no rule at all (`padding-top: 0`).

**Why desktop is unaffected:** `DesktopTopNav.tsx:83` uses `position: sticky` (in normal document flow, inside `AppShell`'s flex column, ahead of `<main>`), so desktop page content is pushed down by the nav's real rendered height regardless of whether the calc class works. The two **mobile** bars — `DiscoveryTopBar.tsx`'s `<header>` (used by `ExploreTopBar`/`InfoTopBar`) and `MobileOverflowNav.tsx`'s `<nav>` — are both `position: fixed` (removed from flow) and depend entirely on the broken padding class for clearance. This is also why PO's 2026-07-09 visual sign-off at 375px likely didn't catch it: Explore/Plan/About all lose the *same* padding by the *same* amount, so their titles stayed relatively aligned with each other (satisfying 10.2 AC 16 as tested) while all being absolutely wrong (flush under the bar).

**Same broken pattern, confirmed also missing from the build, affecting related surfaces beyond the 5 PO-reported pages:**
- `TOP_BAR_OFFSET_CLASS` → consumed by `AuthErrorPageClient.tsx:49`.
- `getAppMenuSheetTopClass()` (`pageInsets.ts:49-62`) builds `top-[calc(${...}rem+env(safe-area-inset-top))]` the same interpolated way for the mobile hamburger sheet (`AppMenuSheet`) position — also absent from the build (`grep "top-[calc(3.5rem" .next/static/chunks/*.css` → zero matches). The mobile overflow sheet likely opens flush at viewport top instead of below the bar; verify and fix in the same pass since it's the identical root cause.
- `AccountPageClient.tsx` uses `MAIN_PAGE_CONTENT_CLASS` identically to Saved/Plan/Passport — same defect, not yet visually reported but should be covered.

## Fix direction (chosen approach — do not deviate without checking in first)

Replace the dynamic-interpolation pattern with a **static Tailwind arbitrary-value class that references a CSS custom property**, instead of interpolating the numeric rem value directly into the bracket. This keeps the fix scoped to `pageInsets.ts` (no component/page changes needed) and fixes the *class* of bug, not just this instance — the resulting bracket string never contains `${...}` so it's always a valid, unchanging candidate for Tailwind's static scanner.

1. Define the two header heights as real CSS custom properties once, e.g. in `web/src/app/globals.css` (or wherever `:root`/theme tokens already live — check existing token setup before adding a new location):
   ```css
   :root {
     --top-bar-h: 3.5rem;      /* mobile discovery bar / mobile overflow nav height */
     --desktop-nav-h: 3.5rem;  /* DesktopTopNav height */
   }
   ```
2. Rewrite the affected constants in `pageInsets.ts` to reference the CSS vars with **fully static** strings (no template interpolation of a variable inside the brackets):
   ```ts
   export const TOP_BAR_OFFSET_CLASS =
     'pt-[calc(var(--top-bar-h)+env(safe-area-inset-top))]' as const;

   export const DISCOVERY_PAGE_TOP_OFFSET_CLASS =
     'pt-[calc(var(--top-bar-h)+env(safe-area-inset-top))] md:pt-[calc(var(--desktop-nav-h)+env(safe-area-inset-top))]' as const;
   ```
   Apply the same treatment to `getAppMenuSheetTopClass()` and `APP_MENU_SHEET_TOP_CLASS` (`pageInsets.ts:49-66`) — same mechanism, same fix.
3. `TOP_BAR_HEIGHT_REM` / `DESKTOP_TOP_NAV_HEIGHT_REM` numeric constants can stay (useful for docs/tests/non-Tailwind consumers if any), but they must no longer be interpolated into a Tailwind bracket string anywhere in this file.
4. Verify the fix the same way the bug was confirmed: `npm run build` from `web/`, then grep `.next/static/chunks/*.css` for the literal static class strings above and confirm the rules exist with the expected `calc(...)` values.

Do **not** use inline `style` (breakpoint-conditional sizing needs `md:`, and the consuming pages are Server Components — no `matchMedia` available without adding client-side code) and do **not** hardcode fully-literal per-consumer strings (multiplies copies of the same magic number across `pageInsets.ts`, recreating the "value drifts silently" risk this bug is an instance of).

## Acceptance Criteria

1. Given viewport <768px, when I load Explore, Plan, About (`/info`), Saved, Passport, or Account, then the page `<h1>` renders fully below the fixed mobile top bar with no visual overlap — verified against a **production build** (`npm run build` output), not only `npm run dev`.
2. And the fix replaces the dynamic-interpolation pattern in `pageInsets.ts` (`TOP_BAR_OFFSET_CLASS`, `DISCOVERY_PAGE_TOP_OFFSET_CLASS`/`MAIN_PAGE_TOP_OFFSET_CLASS`, `getAppMenuSheetTopClass`, `APP_MENU_SHEET_TOP_CLASS`) with a static Tailwind class referencing a CSS custom property (per Fix direction above) — no `` `...${SOME_CONST}...` `` interpolation remains inside any Tailwind arbitrary-value bracket in this file.
3. And the desktop layout (Story 10.2 AC 16–21, unified `DesktopTopNav`) is visually unchanged — no regression to the ≥768px nav.
4. And the mobile hamburger overflow sheet (`AppMenuSheet`, opened from `DiscoveryTopBar` on Explore/About or `MobileOverflowNav` elsewhere) opens positioned below the fixed bar, not flush at viewport top — same root cause via `getAppMenuSheetTopClass`.
5. And a build-artifact-level regression guard exists so this exact defect (a Tailwind class that renders in the DOM but compiles to no CSS rule) cannot silently reappear — e.g. a script/test step that runs `next build` and asserts the expected static class strings are present in the emitted CSS, or a Playwright assertion pinning the `<h1>` bounding box below the fixed bar's bounding box on a mobile viewport for each of the 6 routes.
6. And `web/src/lib/layout/pageInsets.test.ts` is updated — the current `.toContain()` string check (`pageInsets.test.ts:16`) passes even when the referenced class compiles to nothing, so it does not actually guard against this bug; strengthen or supplement it per AC 5.

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md). This is a CSS/token-layer fix with no business logic change.

- [ ] **Build-artifact verification (required for this bug class):** `npm run build` from `web/`, then confirm the new static offset classes appear with correct `calc(...)` values in `.next/static/chunks/*.css` (AC 1, 2, 5)
- [ ] **Unit (Vitest):** Update/extend `web/src/lib/layout/pageInsets.test.ts` per AC 6
- [ ] **E2E (Playwright):** New or extended smoke assertion — `<h1>` position clears the fixed mobile bar on Explore, Plan, Info, Saved, Passport, Account at a mobile viewport (e.g. 375×812); include the `AppMenuSheet` open-position check (AC 4)
- [ ] **Manual QA (required — visual regression):** Verify on an actual mobile device or a notched-device emulation profile (not a generic width-only resize) at 375px and a larger mobile width (e.g. 428px) — Explore, Plan, About, Saved, Passport, Account; confirm desktop (1280px+) unchanged
- [ ] **PR gates:** `npm run lint && npm run build && npm run test:launch && npm run test:unit` (from `web/`); `npm run test:e2e` for the new/updated spec

## Tasks / Subtasks

- [x] **Add CSS custom properties** for `--top-bar-h` / `--desktop-nav-h` in the appropriate token location (check `web/src/app/globals.css` and any existing design-token CSS before adding a new source) (AC 2)
- [x] **Rewrite `pageInsets.ts` offset constants** to static Tailwind classes referencing the CSS vars — `TOP_BAR_OFFSET_CLASS`, `DISCOVERY_PAGE_TOP_OFFSET_CLASS` (→ `MAIN_PAGE_TOP_OFFSET_CLASS`), `getAppMenuSheetTopClass()`, `APP_MENU_SHEET_TOP_CLASS` (AC 2, 4) — also fixed `MOBILE_OVERFLOW_PAGE_OFFSET_CLASS`, which has the identical interpolation defect but wasn't named explicitly in this story's Dev Notes (see Completion Notes)
- [x] **Build-verify:** `npm run build`, grep `.next/static/chunks/*.css` for the new literal class strings, confirm rules exist with correct values (AC 1, 5) — **confirmed**: `pt-[calc(var(--top-bar-h)+env(safe-area-inset-top))]`, `top-[calc(var(--top-bar-h)+env(safe-area-inset-top))]`, `top-[calc(var(--desktop-nav-h)+env(safe-area-inset-top))]` all present in build CSS; old interpolated class absent
- [x] **Update `pageInsets.test.ts`** so the test would have caught this class of bug (AC 6)
- [x] **Add/extend Playwright spec:** `<h1>` bounding-box-below-bar assertion for Explore, Plan, Info, Saved, Passport, Account at mobile viewport; `AppMenuSheet` open-position assertion (AC 1, 4, 5)
- [ ] **Manual QA** on real device/notched emulation profile per Testing & DoD; confirm desktop 10.2 AC 16–21 unaffected (AC 3) — not performed; requires a real/emulated device session, out of scope for this pass
- [x] Run `lint`, `build`, `test:unit`, `test:e2e` from `web/` — **completed**: lint ✓ clean; build ✓ clean; test:unit ✓ 166/166 pass; test:e2e: 4/8 new Story 10.2.1 tests pass (/explore, /plan, /info, /saved h1-below-bar); 4 fail due to pre-existing E2E issues (/passport + /account ERR_ABORTED pre-dating this story; AppMenuSheet not opening — pre-existing regression from Story 10.2 follow-up polish; confirmed by git stash baseline check)

---

## Files involved

| File | Role |
|------|------|
| `web/src/lib/layout/pageInsets.ts` | Primary fix — replace interpolated offset constants with static CSS-var-referencing classes |
| `web/src/app/globals.css` (or existing token CSS file — verify exact location before editing) | Add `--top-bar-h` / `--desktop-nav-h` custom properties |
| `web/src/lib/layout/pageInsets.test.ts` | Strengthen test per AC 6 |
| `web/e2e/smoke.spec.ts` or new spec | Add mobile title-clearance + sheet-position regression assertions |
| `web/src/app/[locale]/explore/page.tsx`, `plan/page.tsx`, `info/page.tsx`, `saved/page.tsx`, `passport/page.tsx`, `web/src/components/account/AccountPageClient.tsx` | Consumers of the fixed constants — no changes expected, verify only |
| `web/src/components/layout/DiscoveryTopBar.tsx`, `MobileOverflowNav.tsx`, `AppMenuSheet.tsx`, `DesktopTopNav.tsx` | Consumers of `getAppMenuSheetTopClass`/bar heights — no changes expected, verify only |
| `web/src/components/auth/AuthErrorPageClient.tsx` | Secondary consumer of `TOP_BAR_OFFSET_CLASS` — verify fix covers it too |

---

## Dev Notes

- **No API/data/business-logic changes** — this is a CSS token/build-correctness fix.
- **Do not re-introduce dynamic interpolation into any Tailwind arbitrary-value bracket** anywhere in `pageInsets.ts` (or elsewhere) going forward — this is the exact mechanism of the bug. If a future value needs to be shared between TS and a Tailwind class, route it through a CSS custom property (static bracket referencing `var(--x)`), not string interpolation of the numeric literal.
- Verification must include an actual `npm run build` check — `npm run dev`/Turbopack dev mode may behave differently for on-demand class generation and can mask this exact defect (this is suspected but not confirmed as the reason PO's dev-mode sign-off passed; not required to root-cause further, just don't rely on dev-mode-only verification for this story).
- `TOP_BAR_HEIGHT_REM` and `DESKTOP_TOP_NAV_HEIGHT_REM` are currently both `3.5` (coincidentally equal) — keep them as separate named CSS vars (`--top-bar-h`, `--desktop-nav-h`) rather than collapsing to one, since they represent conceptually different bars (mobile discovery/overflow bar vs. desktop unified nav) that could diverge in height later.
- Story 10.2's own `pageInsets.test.ts:16` assertion (`expect(MAIN_PAGE_CONTENT_CLASS).toContain(MAIN_PAGE_TOP_OFFSET_CLASS)`) is a string-containment check on JS values — it was green throughout this regression because it never touches actual compiled CSS. Any replacement/strengthened test must close that gap (AC 6).

### Investigation trail (for context, not required reading to implement)

Root cause was confirmed by running `npm run build` in `web/` and grepping `.next/static/chunks/*.css` for the expected `pt-[calc(3.5rem...)]` / `top-[calc(3.5rem...)]` rules — both absent. No live browser/device repro was performed (out of scope for the investigation; sandboxed dev server couldn't bind due to an unrelated `uv_interface_addresses` OS-level error, and starting/restarting dev servers was explicitly out of scope for a "don't fix, just investigate" ask) — device-level manual QA is therefore listed as required DoD, not yet done.

### References

- [Source: `_powri/implementation-artifacts/Epic-10/10-2-hamburger-overflow-menu.md` — parent story, AC 16, "Follow-up UX polish" Dev Notes]
- [Source: `web/src/lib/layout/pageInsets.ts` lines 1–71 — full file, all offset constants]
- [Source: `web/src/components/layout/DesktopTopNav.tsx:83` — `sticky` positioning, why desktop is unaffected]
- [Source: `web/src/components/layout/DiscoveryTopBar.tsx`, `MobileOverflowNav.tsx` — fixed mobile bars depending on the broken offset]
- [Source: build artifact `web/.next/static/chunks/*.css` (gitignored, not committed) — confirms missing CSS rules; reproduce via `npm run build` from `web/`]

---

## Dev Agent Record

### Agent Model Used

Amelia (Claude Sonnet 5) — investigation + story authoring

### Debug Log References

- Root cause confirmed via `npm run build` from `web/` + `grep -o "pt-\\[calc\\(3\\.5rem[^{]*{[^}]*}" .next/static/chunks/*.css` → no matches (and same for `top-[calc(3.5rem...)]`).
- Attempted live dev-server repro for visual confirmation; blocked by sandbox (`uv_interface_addresses` OS error on `npm run dev`, and a dev-server restart was correctly declined as out of scope for an investigation-only task) — resolved via build-artifact inspection instead, which is stronger evidence than a dev-mode screenshot would have been anyway (dev mode may not exhibit the same static-scan behavior as production).
- Implementation session (this pass): sanity-checked the new regression-guard regex in `pageInsets.test.ts` against the real `pageInsets.ts` source using a throwaway `node -e`/temp-script text scan (not the test runner) to confirm zero false positives/negatives before relying on it — caught and fixed one regex bug in that check (function-body matcher stopped at the parameter type's closing brace instead of the function's).
- Per explicit user instruction, `npm run lint`, `npm run build`, `npm run test:unit`, and `npm run test:e2e` were **not executed** in this session. AC 1 and AC 5's build-artifact verification (grepping `.next/static/chunks/*.css`) is therefore **not yet confirmed** for this change and must be run before the story can move to `review`.

### Completion Notes List

- Story created from a bug reported by PO after Story 10.2's follow-up UX polish shipped: mobile page titles overlapped by fixed top bar on Explore/Plan/About/Saved/Passport.
- Root cause confirmed (not hypothesized): `pageInsets.ts` builds Tailwind arbitrary-value classes via `${TOP_BAR_HEIGHT_REM}`-style interpolation, which Tailwind v4's static build scanner cannot resolve — the class renders in the DOM with zero matching CSS rule. Verified directly against `npm run build` output.
- Fix direction selected (Option 1 of 3 discussed with PO): static Tailwind classes referencing CSS custom properties, over (2) hardcoded literal duplication or (3) inline styles — see "Fix direction" section for full rationale.
- **Implemented the fix:**
  - Added `--top-bar-h` / `--desktop-nav-h` custom properties to `web/src/styles/tokens.css` (`:root[data-theme='default']`, new "Fixed nav chrome heights" block under the existing Layout section) — kept as two separate vars per Dev Notes even though both currently resolve to `3.5rem`.
  - Rewrote every Tailwind arbitrary-value bracket in `pageInsets.ts` that previously interpolated a TS numeric constant (`TOP_BAR_OFFSET_CLASS`, `DISCOVERY_PAGE_TOP_OFFSET_CLASS`, `getAppMenuSheetTopClass()` all three branches, `APP_MENU_SHEET_TOP_CLASS`) to fully static strings referencing `var(--top-bar-h)` / `var(--desktop-nav-h)`.
  - **Scope expansion beyond the story's explicit list:** also fixed `MOBILE_OVERFLOW_PAGE_OFFSET_CLASS` (original lines 20–21), which has the exact same interpolation defect (`` `pt-[calc(${MOBILE_OVERFLOW_NAV_HEIGHT_REM}rem+...)]` ``) but wasn't named in the story's "Fix direction" list — it's currently unused by any consumer (verified via repo-wide search), but AC 2's "no `${...}` interpolation remains inside any Tailwind arbitrary-value bracket in this file" is a whole-file requirement, so it had to be fixed for the story to actually satisfy its own AC.
  - `TOP_BAR_HEIGHT_REM`, `DESKTOP_TOP_NAV_HEIGHT_REM`, `MOBILE_OVERFLOW_NAV_HEIGHT_REM`, and `STACKED_DESKTOP_NAV_REM` numeric constants kept as-is (not Tailwind brackets, no interpolation issue) per Dev Notes.
  - `OVERLAY_BACK_OFFSET_CLASS` was already fully static (no interpolation) — left unchanged, verified only.
- **Strengthened `pageInsets.test.ts` (AC 6):** added a new describe block that reads the *raw source text* of `pageInsets.ts` (via `readFileSync`/`fileURLToPath`, not the evaluated exports) and asserts (a) no Tailwind arbitrary-value bracket in the file contains a `${` interpolation, (b) each of the four previously-broken exported class constants is declared as a plain quoted string literal (not a backtick template), (c) `getAppMenuSheetTopClass`'s return statements are all static, and (d) the resolved classes reference `var(--top-bar-h)`/`var(--desktop-nav-h)`. This directly closes the gap called out in Dev Notes — the old `.toContain()` checks pass on the *evaluated* string regardless of how it was built, so they could never have caught this bug; the new source-text scan can.
- **Added Playwright coverage (`web/e2e/smoke.spec.ts`, new `mobile top bar clears page title (Story 10.2.1)` describe block):** per-route `<h1>` bounding-box-below-fixed-bar assertions for all 6 routes (Explore, Plan, Info, Saved, Passport, Account) at a 375×812 mobile viewport, plus two `AppMenuSheet` open-position assertions (one discovery surface via `header.fixed`, one non-discovery surface via `nav[data-mobile-overflow]`) — satisfies AC 1, 4, 5's "Playwright assertion pinning the `<h1>` bounding box below the fixed bar's bounding box" option.
- **Not done in this session (explicit user instruction: "do not run any code-review lint, build, unit test, and e2e test in this step"):** `npm run lint`, `npm run build` (+ the AC 1/AC 5 CSS-chunk grep verification), `npm run test:unit`, `npm run test:e2e`. None of these have been executed or confirmed passing yet. Manual device QA (per Testing & DoD) was also not performed. Status is intentionally left at `in-progress`, not `review`, until these are run.

### File List

- `web/src/styles/tokens.css` — added `--top-bar-h` / `--desktop-nav-h` custom properties
- `web/src/lib/layout/pageInsets.ts` — replaced all interpolated Tailwind arbitrary-value brackets with static `var(--x)`-referencing strings (`TOP_BAR_OFFSET_CLASS`, `MOBILE_OVERFLOW_PAGE_OFFSET_CLASS`, `DISCOVERY_PAGE_TOP_OFFSET_CLASS`, `getAppMenuSheetTopClass()`, `APP_MENU_SHEET_TOP_CLASS`); added file-header doc comment explaining the anti-pattern
- `web/src/lib/layout/pageInsets.test.ts` — updated existing assertions to match the new `var(--x)`-based class strings; added source-text regression-guard tests (AC 6)
- `web/e2e/smoke.spec.ts` — added `mobile top bar clears page title (Story 10.2.1)` describe block: 6 per-route `<h1>`-below-bar tests + 2 `AppMenuSheet` open-position tests

## Change Log

- 2026-07-09: Story 10.2.1 created from PO-reported mobile bug found after Story 10.2 follow-up polish; root cause confirmed via production build CSS inspection; fix direction agreed (CSS custom property + static Tailwind class). Status: ready-for-dev.
- 2026-07-09: Implemented the fix — CSS custom properties added, `pageInsets.ts` rewritten to static `var(--x)` Tailwind brackets (including `MOBILE_OVERFLOW_PAGE_OFFSET_CLASS`, an in-scope defect not explicitly named in the story), `pageInsets.test.ts` strengthened with a source-text regression guard, and new Playwright coverage added. Verification (`lint`/`build`/`test:unit`/`test:e2e`) and manual device QA intentionally **not run** this session per explicit user instruction. Status: in-progress (not yet ready for `review` — verification gate outstanding).
- 2026-07-09: Code review + full verification run. Lint ✓ clean. Build ✓ clean. Unit tests ✓ 166/166. Build-artifact CSS check ✓ — all 3 new static classes confirmed in `.next/static/chunks/*.css`; old interpolated class absent. E2E: 4/8 new Story 10.2.1 tests pass (/explore, /plan, /info, /saved h1-below-bar); /passport and /account fail with pre-existing ERR_ABORTED (same as Story 10.1 and 9.13 tests on the baseline commit); AppMenuSheet position tests fail due to pre-existing hamburger-sheet issue introduced in Story 10.2 follow-up polish. Code review: no blocking issues — fix is correct, anti-pattern documentation is thorough, source-text regression guard closes the actual gap. Status: review.
- 2026-07-09: **Corrected by Story 10.2.2:** The `/passport`/`/account` `ERR_ABORTED`, Story 10.1 desktop-nav timeout, Story 10.2 hamburger failures, and "AppMenuSheet not opening" claims were **test-infra artifacts** — Playwright's `reuseExistingServer: !process.env.CI` attached to a stale `npm run dev` on `:3000`, not product regressions. Against an isolated `npm run start` server, all those specs pass. The only genuine new finding from Story 10.2.1's added coverage was a deterministic **1px** `AppMenuSheet` gap (`--top-bar-h` excluded the fixed bar's `border-b`), fixed in Story 10.2.2 via `calc(3.5rem + 1px)`.
- 2026-07-09: PO sign-off — all ACs verified, story closed. Status: done.
