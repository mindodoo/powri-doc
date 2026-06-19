# Story 1.1: Initialize Next.js Project & Deployment Baseline

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **developer**,
I want a Next.js mobile-first project with Tailwind, TypeScript, and Vercel deployment config,
So that feature work starts on the approved stack with CI-ready builds.

## Acceptance Criteria

1. **Given** no `web/` app exists yet  
   **When** the project is initialized per PRD §7.1 and architecture starter command  
   **Then** `web/` contains Next.js App Router, TypeScript, Tailwind CSS, and ESLint

2. **Given** the initialized project  
   **When** dependencies are installed  
   **Then** `web/package-lock.json` is committed and `npm audit` reports **no high or critical** vulnerabilities (NFR-11)

3. **Given** a production build configuration  
   **When** security headers are inspected in `next.config.ts`  
   **Then** HSTS is set: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload` (PRD §6.2, NFR-3)  
   **And** full CSP is **not** required in this story (Story 1.6 owns CSP)

4. **Given** the running app  
   **When** `GET /api/health` is called  
   **Then** response status is **200** with JSON body `{ "status": "ok" }` for deploy smoke tests

5. **Given** the project root layout  
   **When** `npm run build` runs in `web/`  
   **Then** build completes with zero TypeScript errors

## Tasks / Subtasks

- [x] **Scaffold app** (AC: 1, 5)
  - [x] From repo root, run create-next-app into `web/` (command below)
  - [x] Verify `src/app/` App Router structure and `@/*` import alias
  - [x] Confirm Turbopack dev works: `cd web && npm run dev`
  - [x] Run `npm run build` — must pass

- [x] **Health check route** (AC: 4)
  - [x] Create `web/src/app/api/health/route.ts` — GET returns `{ status: "ok" }`
  - [x] Manually verify `curl localhost:3000/api/health` → 200

- [x] **Security headers baseline** (AC: 3)
  - [x] Add HSTS (and optional `X-Content-Type-Options: nosniff`) via `next.config.ts` `headers()`
  - [x] Do **not** add full CSP yet — Story 1.6

- [x] **Dependency hygiene** (AC: 2)
  - [x] Commit `package-lock.json`
  - [x] Run `npm audit` — fix or document any high/critical (must be clean to pass AC)

- [x] **Vercel readiness** (AC: 1, 4)
  - [x] Add `web/README.md` with dev/build commands and note that app root is `web/`
  - [x] Extend generated `web/AGENTS.md` with pointers: `architecture.md`, `DESIGN.md`, `EXPERIENCE.md`, `docs/tracking-plan.md`
  - [x] Optional: root-level note in repo `README.md` if missing — "App code lives in `web/`"

- [x] **Out of scope — do NOT implement in this story**
  - Design tokens (Story 1.2)
  - Content loader (Story 1.3)
  - BottomNav / routes beyond placeholder (Story 1.4)
  - Analytics / consent (Story 1.5)
  - CSP / rate limiting scaffold (Story 1.6)
  - i18n (Story 1.7)

## Dev Notes

### Critical constraints

| Rule | Detail |
|------|--------|
| **App location** | All Next.js code lives in **`web/`** — NOT repo root. Keeps `content/`, `docs/`, `_bmad-output/` sibling to app. |
| **Import alias** | Use `@/*` → `src/*` only. No deep relative imports like `../../../`. |
| **No secrets** | Do not add API keys. No `.env.local` with real values in git. |
| **Brownfield repo** | Resort markdown already exists at `content/resorts/`. Do not move or duplicate content into `web/`. |
| **Scope boundary** | This story is **scaffold + health + HSTS + audit**. Resist adding features from later stories. |

### Initialization command (exact)

Run from **`/Users/mindodoo/Projects/japan_winter_sport`** (repo root):

```bash
npx create-next-app@latest web \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-npm \
  --yes
```

**Expected stack (June 2026):** Next.js 16.x, Tailwind v4 (bundled with template), Turbopack default for dev.

### Project structure after this story

```
japan_winter_sport/
├── content/              # existing — do not touch
├── docs/                 # existing
├── web/                  # NEW — this story creates
│   ├── AGENTS.md         # extend with project pointers
│   ├── README.md
│   ├── package.json
│   ├── package-lock.json
│   ├── next.config.ts    # HSTS headers here
│   ├── tsconfig.json
│   ├── tailwind.config.ts (or postcss — per template)
│   └── src/
│       └── app/
│           ├── layout.tsx
│           ├── page.tsx      # default starter page OK for now
│           ├── globals.css
│           └── api/
│               └── health/
│                   └── route.ts
```

Full target structure is in [architecture.md](../planning-artifacts/architecture.md) — implement only what this story requires.

### HSTS implementation pattern

Add to `web/next.config.ts`:

```typescript
const nextConfig = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=31536000; includeSubDomains; preload',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};

export default nextConfig;
```

Vercel also enforces HTTPS at edge; explicit HSTS satisfies PRD §6.2 for direct header inspection.

### Health route pattern

```typescript
// web/src/app/api/health/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ status: 'ok' });
}
```

Architecture defines this route for all deploy smoke tests [architecture.md § API & Communication](../planning-artifacts/architecture.md).

### Architecture compliance

- **Starter template:** Matches architecture "Selected Starter" section exactly.
- **Deployment:** Vercel-ready; no `vercel.json` required for MVP unless monorepo root detection needs `web/` as root directory — set **Root Directory = `web`** in Vercel project settings.
- **CI-ready:** Story enables future GitHub Actions (`npm ci && npm run lint && npm run build && npm audit`) — workflow file is IR gap, not required here.

### Library / framework requirements

| Package | Purpose | Notes |
|---------|---------|-------|
| `next` | Framework | App Router only — no Pages Router |
| `react`, `react-dom` | UI | Bundled with Next |
| `typescript` | Types | strict mode default |
| `tailwindcss` | Styling | v4 via template |
| `eslint`, `eslint-config-next` | Lint | Run `npm run lint` before marking done |

**Do not add yet:** `zustand`, `gray-matter`, `posthog-js`, `@anthropic-ai/sdk`, `next-intl` — later stories.

### Testing requirements

- **Manual:** `npm run dev` → app loads; `/api/health` → 200
- **Manual:** `npm run build` → success
- **Manual:** `npm audit` → no high/critical
- **No automated test files required** for this scaffold story

### UX / design

Not applicable for scaffold — default Next.js starter page is acceptable. Story 1.2 applies Theme A tokens.

### Epic 1 context (cross-story)

This is the **first of 7** foundation stories. Subsequent stories depend on `web/` existing:

| Story | Builds on 1.1 |
|-------|----------------|
| 1.2 | `globals.css`, fonts |
| 1.3 | `src/lib/content/` |
| 1.4 | routes + BottomNav |
| 1.5 | analytics providers |
| 1.6 | CSP + `/api/` scaffold expansion |
| 1.7 | `messages/en.json` + i18n |

### Previous story intelligence

None — greenfield app code. Content layer is brownfield (`content/resorts/*.md` all `draft` status).

### Git intelligence

No prior app commits expected. If `web/` partially exists, verify against AC before re-scaffolding.

### Latest tech notes (June 2026)

- `create-next-app@latest` defaults: TypeScript, Tailwind, ESLint, App Router, Turbopack, `AGENTS.md` for coding agents.
- Use `@latest` at init time — do not pin old Next 14/15 unless build fails.
- Tailwind v4 may use `@import "tailwindcss"` in CSS instead of v3 `@tailwind` directives — follow generated template; do not downgrade.

### Project context reference

- [project-context.md](../../project-context.md) — brownfield content / greenfield app
- [architecture.md](../planning-artifacts/architecture.md) — authoritative stack decisions
- [epics.md](../planning-artifacts/epics.md) — Story 1.1 AC source
- [implementation-readiness-report-2026-06-12.md](../planning-artifacts/implementation-readiness-report-2026-06-12.md) — READY WITH MINOR GAPS

### References

- [Source: japan_ski_prd.md §7.1 Frontend](../../japan_ski_prd.md)
- [Source: japan_ski_prd.md §6.2 HSTS](../../japan_ski_prd.md)
- [Source: architecture.md § Starter Template Evaluation](../planning-artifacts/architecture.md)
- [Source: architecture.md § Project Structure](../planning-artifacts/architecture.md)
- [Source: epics.md § Story 1.1](../planning-artifacts/epics.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — Next.js 16.2.9, zero TS errors
- `npm run lint` — clean
- `npm audit --audit-level=high` — 2 moderate (postcss via next); no high/critical
- `curl localhost:3000/api/health` → `{"status":"ok"}` HTTP 200

### Completion Notes List

- Scaffolded `web/` with `create-next-app@16.2.9` (Next.js 16.2.9, Tailwind v4, Turbopack dev)
- Added HSTS + `X-Content-Type-Options` in `next.config.ts`
- Added `GET /api/health` smoke-test route
- Updated `web/README.md` and extended `web/AGENTS.md` with architecture/UX/tracking pointers
- Added root `README.md` noting app lives in `web/`
- Repo root has no git (`baseline_commit: NO_VCS`); `create-next-app` initialized nested git inside `web/` only

### File List

- `web/` (scaffold — package.json, package-lock.json, tsconfig.json, eslint config, src/app/*, etc.)
- `web/next.config.ts` (modified — security headers)
- `web/src/app/api/health/route.ts` (new)
- `web/README.md` (modified)
- `web/AGENTS.md` (modified)
- `README.md` (new — repo root)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve  
**Layers:** Blind Hunter, Edge Case Hunter, Acceptance Auditor

### Summary

All five acceptance criteria satisfied. Scaffold matches architecture starter command; health route and HSTS headers correct; build/lint pass; audit shows no high/critical vulnerabilities.

### Action Items

- [x] [Review][Defer] Nested `.git` inside `web/` from create-next-app — defer; remove when repo root is initialized
- [x] [Review][Defer] Starter `page.tsx` still uses Geist/zinc/dark classes — deferred to Story 1.2 (design tokens)

### Review Findings

- [x] [Review][Defer] Nested git in `web/` — pre-existing scaffold artifact, not blocking AC
- [x] [Review][Defer] `globals.css` dark-mode override — Story 1.2 replaces with Theme A tokens
