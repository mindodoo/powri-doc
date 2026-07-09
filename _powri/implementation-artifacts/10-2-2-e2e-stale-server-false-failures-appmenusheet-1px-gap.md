---
baseline_commit: c15a7294e8ab531e30007a00801b8730088f6ef4
---

# Story 10.2.2: E2E "Pre-Existing Failure" Claims Are Test-Infra Artifacts, Not Product Bugs — Plus a Real 1px `AppMenuSheet` Gap

Status: done

**Epic:** 10 · **Parent:** Follow-up to Story [10.2.1](./10-2-1-mobile-top-bar-title-overlap.md) verification session · **Shard:** [`epic-10-nav-saved.md`](../planning-artifacts/epics/phase2/shards/epic-10-nav-saved.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.
**Reported:** 2026-07-09 (PO noticed the "pre-existing E2E failure" claims in 10.2.1's Dev Agent Record didn't match production behaviour, and separately hit the same symptoms live on `localhost`) · **Environment:** Local dev (`npm run dev`) and local E2E (`npm run test:e2e`) only — **not reproducible in production**
**Severity:** P2 — no user-facing product bug for the ERR_ABORTED claim (test-infra false positive that risks masking real regressions behind an incorrect "pre-existing" label); P3 cosmetic 1px gap for the genuine `AppMenuSheet` finding

---

## Story

As a developer relying on the E2E suite to gate PRs,
I want `/passport` and `/account` navigation and the `AppMenuSheet` position assertions to reflect the real, isolated-build behaviour of the app — not whatever server happens to already be listening on port 3000,
so that "pre-existing failure" labels in story Dev Agent Records are trustworthy and real regressions aren't hidden behind them.

---

## Background — why this story exists

Story 10.2.1's Dev Agent Record (verification session, 2026-07-09) claimed 4 of its 8 new E2E assertions failed due to **pre-existing** issues unrelated to that story's CSS fix:

1. `page.goto('/passport')` and `page.goto('/account')` → `net::ERR_ABORTED`, said to pre-date the story (blamed on Story 10.1 / 9.13 tests) and "confirmed by git stash baseline check."
2. Both new `AppMenuSheet opens below the fixed bar` assertions failing with "sheet not found" / not opening, said to be a regression from Story 10.2's follow-up UX polish.

The PO flagged two problems with this: **(a)** neither symptom is visible in production — only the 10.2.1 CSS/title-overlap bug was — and **(b)** the PO is hitting ERR_ABORTED-like slowness on `/account`/`/passport` on `localhost` *right now*, which doesn't square with "pre-existing since two stories ago and already tracked."

This story is the requested investigation writeup + fix. **Investigation is complete and both root causes are confirmed with reproducible evidence below. No code changes have been made yet** — per explicit instruction, implementation is deferred to a future dev pass on this story.

---

## Root cause 1 (Confirmed): `/passport` + `/account` "ERR_ABORTED" is `reuseExistingServer` attaching to a stale/overloaded server, not a route bug

### Evidence

**A. The real dev server, observed live, hangs specifically on `/account`, `/passport`, `/auth/error`:**

The user's actual `npm run dev` server (terminal log, `web/` cwd) shows:

```
GET /saved 200 in 41ms
GET /account 200 in 33.8s
GET /account 200 in 33.8s
...
GET /auth/error?error_code=access_denied 200 in 35.0s
GET /passport 200 in 35.0s
GET /account 200 in 35.0s
GET /passport 200 in 34.8s
GET /account 200 in 35.0s
GET /passport 200 in 17.4min
```

Meanwhile `/`, `/explore`, `/saved`, `/plan`, `/info` consistently respond in 30–300ms in the same log, interleaved between the slow requests — ruling out a global cause (middleware, network egress, etc.) that would affect every route equally.

**B. The same three routes are instant on a freshly-started, isolated production server:**

Killed nothing, just started a second `next start` on a free port (3012) with the same CI-style env vars used by `playwright.config.ts`, and hit every route Story 10.2.1 tests:

```
/         -> http_code=200 time=0.168s   (cold start)
/explore  -> http_code=200 time=0.012s
/saved    -> http_code=200 time=0.007s
/plan     -> http_code=200 time=0.007s
/info     -> http_code=200 time=0.009s
/passport -> http_code=200 time=0.007s
/account  -> http_code=200 time=0.007s
```

All 7 routes, including `/passport` and `/account`, are sub-10ms. **The route code has no bug.**

**C. Re-running the actual failing Playwright specs against that same clean, isolated server — no stash, no code changes, current working tree as-is:**

```
✓ /explore h1 renders fully below the fixed mobile top bar (496ms)
✓ /plan h1 renders fully below the fixed mobile top bar (170ms)
✓ /info h1 renders fully below the fixed mobile top bar (120ms)
✓ /saved h1 renders fully below the fixed mobile top bar (128ms)
✓ /passport h1 renders fully below the fixed mobile top bar (143ms)     ← previously reported ERR_ABORTED
✓ /account h1 renders fully below the fixed mobile top bar (203ms)     ← previously reported ERR_ABORTED
✓ desktop top nav replaces bottom nav and reaches passport stub (3.5s) ← Story 10.1, previously reported as pre-existing failure ("subtree intercepts pointer events" / 6.3min timeout)
✓ mobile hamburger on saved page shows Passport, About, Sign in (800ms) ← Story 10.2, previously reported as pre-existing failure
✓ mobile discovery hamburger shows About not Home or Explore (830ms)  ← Story 10.2, previously reported as pre-existing failure
```

**All of these pass** against a clean server. Only the two brand-new 10.2.1 `AppMenuSheet` position assertions still fail — see Root cause 2 below, which is a real (tiny) bug, not this infra issue.

### Why this happens

`web/playwright.config.ts`:

```16:43:web/playwright.config.ts
export default defineConfig({
  ...
  webServer: {
    command: `npm run start -- -p ${PORT}`,
    url: baseURL,
    reuseExistingServer: !process.env.CI,
    ...
  },
});
```

Locally (`CI` unset), `reuseExistingServer: true` means: **if anything is already listening on port 3000, Playwright silently attaches to it and never runs `npm run start` at all.** During active development, something is *almost always* already listening on 3000 — typically the developer's own `npm run dev` process (confirmed running, pid 25711, in this session).

Next.js dev mode lazily compiles route bundles on first visit and evicts rarely-visited ones from its on-demand-entries cache after a period of inactivity, forcing a full recompile (including the entire Supabase/Auth dependency graph those three routes pull in) on the next hit. `/account`, `/passport`, and `/auth/error` are visited far less often than `/explore`/`/saved`/`/plan`/`/info` in a typical manual dev session (confirmed by hit-count in the terminal log), so they're disproportionately likely to be cold — explaining both the ~33–35s stalls and the one 17.4-minute outlier (compounded by concurrent load). This class of stall exceeds Playwright's default 30s navigation timeout, which Playwright/Chromium reports as `net::ERR_ABORTED; maybe frame was detached?` when it cancels the in-flight navigation.

This **cannot happen in production**: Vercel serverless functions are precompiled ahead of time with no dev-mode on-demand-entries behavior, so the user never sees it there — consistent with the PO's report.

It also explains why the previous agent's `git stash` baseline check "confirmed" the failure was pre-existing: `git stash` rewrites files on disk but does **not** restart an already-running Node process. Whatever stale/cold server `reuseExistingServer` had already attached to before the stash remained running, unrestarted, *through* the stash — so the "baseline" run and the "current" run were silently hitting the exact same already-affected server both times. The comparison never actually isolated the two code states from each other.

### Fix direction (not yet implemented — proposed for the dev pass)

1. In `web/playwright.config.ts`, do not rely on `reuseExistingServer: !process.env.CI` for correctness-sensitive local runs. Options to evaluate:
   - Always use a dedicated, non-default port for E2E (e.g. via `PLAYWRIGHT_PORT`) so it never collides with a developer's ambient `npm run dev`/`npm run start` on 3000, and document this in `docs/process/testing-strategy.md`.
   - Or set `reuseExistingServer: false` unconditionally (always spin up + tear down a fresh `npm run start`), accepting the extra build/start cost locally, if isolation is valued over speed.
2. Document in `docs/process/testing-strategy.md` (or an dev-agent guardrail) that a `git stash`-based "baseline check" against a *locally reused* server is not a valid way to distinguish pre-existing failures from regressions — it must either (a) run against a guaranteed-fresh isolated server (dedicated port, `reuseExistingServer: false`), or (b) compare against CI's last known-good run instead.
3. Re-run the full E2E suite once (1)–(2) land, to get an honest current baseline and correct the "pre-existing" labels on Stories 10.1, 10.2, 9.13's Dev Agent Records if those specs also turn out to pass cleanly (see Root cause 1's evidence — the Story 10.1 and 10.2 specs re-tested clean above, so those stories' "pre-existing failure" notes are very likely already stale and should be revisited too, though re-confirming that is out of this story's direct scope unless trivial to fold in).

---

## Root cause 2 (Confirmed): `AppMenuSheet` sits exactly 1px above the fixed bar's true bottom edge

### Evidence

Re-running Story 10.2.1's own two new assertions against the clean, isolated server (no stash, no infra confound) — deterministic across repeated runs, not flaky:

```
AppMenuSheet opens below the fixed bar on a discovery surface (Explore)
  Expected: >= 57
  Received:    56

AppMenuSheet opens below the fixed bar on a non-discovery surface (Saved)
  Expected: >= 57
  Received:    56
```

The sheet's top edge is consistently **exactly 1px** above the bar's bottom edge — not "not opening," not "position broken," not the dramatic click-interception failure described in 10.2.1's Dev Agent Record (that description traces back to Root cause 1's stale-server confound, per the clean re-run above).

### Why

`--top-bar-h: 3.5rem` (56px) is used both for the bar's own height (`h-14` = 3.5rem = 56px) **and** for the `AppMenuSheet`'s `top` offset via `getAppMenuSheetTopClass()`. But every fixed bar that uses this height also has a `border-b border-divider` (1px) stacked on top of it:

```62:64:web/src/components/layout/MobileOverflowNav.tsx
className="fixed inset-x-0 top-0 z-[39] border-b border-divider bg-surface pt-[env(safe-area-inset-top)] md:hidden"
```

(and identically for `ExploreTopBar` / `InfoTopBar`'s `headerClassName`, both `border-b border-divider`). The bar's real rendered height is therefore **56px content + 1px border = 57px**, but `top: calc(var(--top-bar-h) + env(safe-area-inset-top))` only offsets by 56px — one pixel short of the bar's true bottom edge, so the sheet's top edge lands 1px inside (behind) the bar's border.

### Fix direction (not yet implemented — proposed for the dev pass)

Prefer accounting for the border in the shared height var, since `--top-bar-h`/`--desktop-nav-h` are documented as "mirrors the bar height" and the border is part of every consuming bar's actual rendered footprint:

```css
--top-bar-h: calc(3.5rem + 1px); /* h-14 (56px) + border-b (1px) */
--desktop-nav-h: calc(3.5rem + 1px); /* h-14 (56px) + border-b (1px), if DesktopTopNav also has border-b */
```

Verify `DesktopTopNav.tsx`'s border situation before changing `--desktop-nav-h` — this story's investigation only measured the two mobile bars pixel-for-pixel; confirm whether the desktop nav has the same `border-b` before assuming the identical +1px correction applies there too. Re-run the build-artifact CSS check (per 10.2.1's precedent) after changing the var, since `calc(3.5rem + 1px)` is still a fully static value with no interpolation risk.

Do not "fix" this by loosening the test assertion (e.g. `>= barBox!.y + barBox!.height - 1`) — that would mask the actual 1px visual gap/overlap rather than correct it.

---

## Acceptance Criteria

1. `web/playwright.config.ts` (or equivalent test-runner setup) guarantees local E2E runs against an isolated server unaffected by whatever else is listening on the default dev port — either a dedicated non-3000 port, or `reuseExistingServer: false`, with the choice documented in `docs/process/testing-strategy.md`.
2. Re-running the full local E2E suite (`npm run test:e2e`) after AC 1 shows `/passport` and `/account` navigation succeeding with no `ERR_ABORTED`, and the Story 10.1/10.2 specs previously mislabeled "pre-existing" passing cleanly (matching the clean-server evidence already gathered in this story).
3. `--top-bar-h` (and `--desktop-nav-h`, if `DesktopTopNav` also has a `border-b`) in `web/src/styles/tokens.css` account for the fixed bar's real rendered height including its border, so `AppMenuSheet`'s computed `top` lands exactly at (not 1px above) the bar's true bottom edge.
4. Story 10.2.1's two `AppMenuSheet opens below the fixed bar` E2E assertions (`web/e2e/smoke.spec.ts`) pass against a freshly isolated server, with the existing `>=` assertion unchanged (not loosened).
5. Story 10.2.1's Dev Agent Record is corrected (via a dated Change Log entry, not a rewrite of history) to reflect that the `/passport`/`/account` ERR_ABORTED and `AppMenuSheet`-not-opening failures were test-infra artifacts, not pre-existing product regressions from Stories 10.1/10.2/9.13 — so those stories' own records aren't left carrying an incorrect regression note.
6. No regression to Story 10.2.1's already-passing AC (h1-below-bar on all 6 routes) or to Story 10.2's desktop nav (AC 16–21).

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../docs/process/testing-strategy.md).

- [x] **E2E isolation fix verified:** confirm (e.g. via a deliberately-left-running stray server during a test run) that the new config can no longer silently attach to it
- [x] **E2E (Playwright):** full `npm run test:e2e` run, zero failures attributable to server staleness
- [x] **Build-artifact verification:** if `--top-bar-h`/`--desktop-nav-h` change, re-run the `npm run build` + `.next/static/chunks/*.css` grep check from Story 10.2.1 to confirm the new `calc()` value compiles correctly
- [x] **Manual QA:** visually confirm the hamburger sheet has zero gap/overlap against the fixed bar's bottom border on a real mobile viewport, both discovery (Explore) and non-discovery (Saved) surfaces
- [x] **PR gates:** `npm run lint && npm run build && npm run test:launch && npm run test:unit` (from `web/`); `npm run test:e2e`

---

## Tasks / Subtasks

- [x] **Investigate root causes** (AC —) — **done, this session; see Root cause 1 & 2 above with reproducible evidence.** No code changed.
- [x] Update `web/playwright.config.ts` webServer config per Root cause 1's fix direction (AC 1)
- [x] Document the corrected local-baseline-check methodology in `docs/process/testing-strategy.md` (AC 1)
- [x] Re-run full local E2E suite against the fixed config; confirm `/passport`/`/account`/Story 10.1/10.2 specs pass clean (AC 2)
- [x] Check `DesktopTopNav.tsx` for `border-b`; update `--top-bar-h` and (if applicable) `--desktop-nav-h` in `web/src/styles/tokens.css` to include the border (AC 3)
- [x] Re-run Story 10.2.1's `AppMenuSheet` position assertions; confirm both pass without loosening the assertion (AC 4)
- [x] Add a dated Change Log entry to Story 10.2.1 correcting the "pre-existing failure" characterization (AC 5)
- [x] Run `lint`, `build`, `test:unit`, `test:e2e` from `web/` (AC 6, DoD)

---

## Files involved

| File | Role |
|------|------|
| `web/playwright.config.ts` | Primary fix — E2E server isolation (Root cause 1) |
| `docs/process/testing-strategy.md` | Document corrected baseline-check methodology |
| `web/src/styles/tokens.css` | `--top-bar-h` / `--desktop-nav-h` +1px border correction (Root cause 2) |
| `web/e2e/smoke.spec.ts` | `AppMenuSheet` position assertions (Story 10.2.1) — verify only, do not loosen |
| `web/src/components/layout/DesktopTopNav.tsx` | Verify border-b presence before changing `--desktop-nav-h` |
| `web/src/components/layout/MobileOverflowNav.tsx`, `web/src/components/explore/ExploreTopBar.tsx`, `web/src/components/info/InfoTopBar.tsx` | Source of the confirmed 1px `border-b` on the mobile fixed bars |
| `_powri/implementation-artifacts/10-2-1-mobile-top-bar-title-overlap.md` | Add corrective Change Log entry (AC 5) |

---

## Dev Notes

- **This story is investigation + fix-direction only as of creation.** Per explicit instruction, no code has been changed — the "Fix direction" sections above are recommendations for the next dev pass, not applied changes.
- **Do not re-run a `git stash`-based baseline check the same way again** without first confirming no server is already listening on the port under test (`lsof -nP -iTCP:3000 -sTCP:LISTEN` or equivalent) — that was the direct cause of the false "pre-existing" conclusion this story corrects.
- The 1px `AppMenuSheet` gap (Root cause 2) is unrelated to Root cause 1 and was only visible once Root cause 1's confound was removed — it's a genuinely new, very minor finding from Story 10.2.1's own added test coverage, not a "Story 10.2 regression" as originally claimed.
- Desktop's `AppMenuSheet` variant was not verified against the same pixel-perfect check in this investigation (Story 10.2.1 added no desktop-variant position assertion) — worth a quick look during implementation given `DesktopTopNav` may or may not share the same `border-b` pattern.

### Investigation trail

- Confirmed a real `npm run dev` process was listening on port 3000 during this investigation (`lsof -nP -iTCP:3000 -sTCP:LISTEN` → pid 25711); its terminal log showed `/account`/`/passport`/`/auth/error` GETs taking 33.8s–17.4min while `/explore`/`/saved`/`/plan`/`/info` stayed under 300ms in the same log window.
- Started a second, fully isolated `next start` on port 3012 with the same CI-style env vars as `playwright.config.ts`; all 7 routes (`/`, `/explore`, `/saved`, `/plan`, `/info`, `/passport`, `/account`) responded in 7–170ms via `curl`.
- Re-ran the exact Playwright specs previously reported as "pre-existing failures" (Story 10.1's `desktop top nav replaces bottom nav and reaches passport stub`, Story 10.2's two hamburger specs, Story 10.2.1's 6 h1-below-bar specs) against that clean port-3012 server with `PLAYWRIGHT_PORT=3012` (no `CI` env, so `reuseExistingServer: true` correctly reused the already-clean server) — all passed. Only Story 10.2.1's 2 new `AppMenuSheet opens below the fixed bar` specs still failed, deterministically, by exactly 1px (`Received: 56`, `Expected: >= 57`) across repeated runs.
- Traced the 1px gap to `--top-bar-h: 3.5rem` (56px) excluding the 1px `border-b` present on every consuming fixed bar (`MobileOverflowNav`, `ExploreTopBar`, `InfoTopBar`), confirmed by reading each component's className directly.

### References

- [Source: `_powri/implementation-artifacts/10-2-1-mobile-top-bar-title-overlap.md` — Dev Agent Record, Debug Log References, and Change Log entries claiming the pre-existing failures this story corrects]
- [Source: `web/playwright.config.ts` — `reuseExistingServer: !process.env.CI`]
- [Source: `web/src/lib/layout/pageInsets.ts` lines 60–74 — `getAppMenuSheetTopClass`]
- [Source: `web/src/styles/tokens.css` lines 93–97 — `--top-bar-h` / `--desktop-nav-h`]
- [Source: `web/src/components/layout/MobileOverflowNav.tsx`, `web/src/components/explore/ExploreTopBar.tsx`, `web/src/components/info/InfoTopBar.tsx` — `border-b border-divider` on every fixed bar consuming `--top-bar-h`]

---

## Dev Agent Record

### Agent Model Used

Amelia (Claude Sonnet 5) — investigation only, this session; implementation pass (Composer)

### Debug Log References

- `DesktopTopNav.tsx` verified: no `border-b` on the nav chrome (`sticky top-0` header only); `--desktop-nav-h` left at `3.5rem`, only `--top-bar-h` bumped to `calc(3.5rem + 1px)`.
- Build-artifact check: `grep -roh "calc(3.5rem + 1px)" .next/static/chunks/*.css` → match confirmed after `npm run build`.
- E2E isolation verified with ambient `npm run dev` still listening on `:3000` (pid 25711) — Playwright started fresh server on `:3012` and both `AppMenuSheet` specs passed.
- Full suite: 22/25 pass; 3 `account-avatar-upload` failures are Supabase session mint errors (`Failed to create E2E user`) — unrelated to server staleness (Story 9.13 requires real `SUPABASE_SERVICE_ROLE_KEY`).

### Completion Notes List

- Investigated at PO's request after Story 10.2.1's Dev Agent Record claimed 4 "pre-existing" E2E failures that don't reproduce in production and that the PO is independently hitting on `localhost`.
- Confirmed via direct reproduction (not re-reading old logs) that the `/passport`/`/account` `ERR_ABORTED` claim and the dramatic `AppMenuSheet`-not-opening claim are both artifacts of Playwright's `reuseExistingServer: !process.env.CI` attaching to a stale/cold `npm run dev` process rather than a fresh, isolated `npm run start` build — not real product bugs, and not actually "pre-existing" in the sense of being present in the shipped Story 10.1/10.2/9.13 code.
- Found one genuine, small, previously-mischaracterized bug: the two new `AppMenuSheet` position assertions added by Story 10.2.1 itself fail by a deterministic 1px, caused by `--top-bar-h` excluding the fixed bar's `border-b`.
- **No code changes made in investigation session** — investigation and story creation only, per explicit instruction not to start fixing.
- **Implementation pass:**
  - `playwright.config.ts`: default port `3012` (avoids ambient `:3000` dev server); `reuseExistingServer: false` unconditionally (always fresh `npm run start`).
  - `docs/process/testing-strategy.md`: documented isolation behaviour and invalid `git stash` baseline methodology.
  - `tokens.css`: `--top-bar-h: calc(3.5rem + 1px)` for mobile fixed bars' `border-b`; `--desktop-nav-h` unchanged (no border on desktop nav).
  - Verification: lint ✓, build ✓, unit 166/166 ✓, story-related E2E 14/14 ✓ (all h1-below-bar, AppMenuSheet, Story 10.1/10.2 specs).

### File List

- `web/playwright.config.ts` — E2E server isolation (port 3012 default, `reuseExistingServer: false`)
- `docs/process/testing-strategy.md` — local E2E isolation + invalid baseline-check guardrail
- `web/src/styles/tokens.css` — `--top-bar-h` includes 1px `border-b`
- `_powri/implementation-artifacts/10-2-1-mobile-top-bar-title-overlap.md` — corrective Change Log entry (AC 5)
- `_powri/implementation-artifacts/sprint-status.yaml` — status tracking

## Change Log

- 2026-07-09: Story created from PO-requested investigation into Story 10.2.1's "pre-existing E2E failure" claims. Both claimed root causes reproduced and diagnosed: (1) `/passport`/`/account` `ERR_ABORTED` and the Story 10.1/10.2 "regressions" are test-infra artifacts of `reuseExistingServer` attaching to a stale dev server, confirmed absent against a clean isolated build; (2) `AppMenuSheet` has a genuine, minor 1px positioning gap from `--top-bar-h` excluding the fixed bar's `border-b`, unrelated to the originally-claimed cause. Status: ready-for-dev — implementation intentionally deferred per explicit "do not start fixing yet" instruction.
- 2026-07-09: Implemented fixes — Playwright isolation (port 3012 + `reuseExistingServer: false`), `--top-bar-h` border correction, testing-strategy documentation, Story 10.2.1 Change Log correction. All story-related E2E specs pass on isolated server. Status: review.
- 2026-07-09: PO sign-off — all ACs verified, story closed. Status: done.
