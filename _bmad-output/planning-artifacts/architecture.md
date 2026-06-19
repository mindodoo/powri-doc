---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments:
  - japan_ski_prd.md
  - _bmad-output/planning-artifacts/epics.md
  - _bmad-output/planning-artifacts/ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-japan_winter_sport-2026-06-12/EXPERIENCE.md
  - content/CONTENT_MODEL.md
  - docs/tracking-plan.md
  - project-context.md
workflowType: architecture
project_name: japan_winter_sport
user_name: Mindodoo
date: 2026-06-12
lastStep: 8
status: complete
completedAt: 2026-06-12
---

# Architecture Decision Document

**Japan Ski & Snowboard Trip Planner** — Phase 1 foundation architecture for consistent AI-agent implementation.

**Companion artifacts:**
- PRD v1.1 (`japan_ski_prd.md`)
- UX spines (`ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md`, `EXPERIENCE.md`)
- Epics (`epics.md` — 5 epics, 27 stories)
- Analytics (`docs/tracking-plan.md`)

---

## Project Context Analysis

### Requirements Overview

**Functional Requirements (Phase 1):**

| FR | Architectural implication |
|----|---------------------------|
| FR-1 | Client-side filter/search over build-time resort index; shared list component for Home + Explore |
| FR-2 | SSG/ISR resort detail pages from markdown; image attribution component; trail map as external link |
| FR-3 | Getting There as section or sub-route; structured transport/gateway data from frontmatter |
| FR-4 | Quiz state machine + scoring service (client-side Phase 1); filtered list view |
| FR-5 | Onboarding flag in `localStorage`; no server persistence Phase 1 |
| FR-6 | Serverless `/api/chat` proxy; seed-content retrieval layer; rate limits + cost guardrails |
| FR-7 | Static Info + legal pages; minimal SSR |

**Non-Functional Requirements:**

| NFR | Architectural driver |
|-----|---------------------|
| Mobile-first 375–430px | Tailwind breakpoints; bottom nav layout shell |
| SSG/ISR performance | Build-time markdown import; edge cache on Vercel |
| Security §6 | CSP headers, API key server-only, rate limiting middleware |
| GDPR/PDPA | Consent-gated PostHog; anonymous default |
| No DB Phase 1 | File-based content; client persistence only |
| AI cost bounds §4.4 | Session caps, IP limits, kill-switch env flag |
| Analytics §13 | PostHog EU + Plausible; event helper module |
| Unsplash §17.2 | Image metadata in content; attribution component |

**Scale & Complexity:**

- **Primary domain:** Full-stack mobile web (Next.js App Router)
- **Complexity level:** Medium — no multi-tenancy, no real-time collab; bounded AI integration
- **Estimated architectural components:** ~12 (content loader, resort index, quiz engine, chat proxy, analytics, consent, i18n shell, design tokens, nav shell, image service, rate limiter, security headers)

### Technical Constraints & Dependencies

- **Content source of truth:** Git markdown in `content/resorts/` — build-time import only (no CMS Phase 1)
- **Brownfield content / greenfield app:** 20 resort files exist; application code does not
- **Solo + AI build:** Architecture must be explicit enough that multiple agent sessions produce compatible code
- **Deployment target:** Vercel (aligned with Next.js serverless)
- **External APIs (Phase 1):** Anthropic Claude, PostHog, Plausible, Unsplash (build/dev only until app approved)
- **Deferred Phase 2+:** Supabase auth, Google Places, live snow search, Plan tab persistence

### Cross-Cutting Concerns Identified

1. **Content pipeline** — parse, validate, filter `content_status: published`, expose typed resort model
2. **Design tokens** — CSS variables from `DESIGN.md`; Tailwind theme extension
3. **Analytics & consent** — every feature wires through shared `track()` helper
4. **Security** — CSP, CORS, rate limits on `/api/*`
5. **AI guardrails** — shared middleware for caps, kill-switch, seed-first retrieval
6. **i18n scaffold** — string extraction; EN-only ship
7. **Image attribution** — Unsplash metadata → reusable component (R1: abstract provider interface)
8. **Season currency** — T2 field labelling in content types + UI formatters

---

## Starter Template Evaluation

### Primary Technology Domain

Mobile-first **full-stack web application** — Next.js App Router with TypeScript, Tailwind CSS, and serverless API routes.

PRD §7.1 and Epic 1 Story 1.1 specify this stack. User skill level: intermediate.

### Starter Options Considered

| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **create-next-app@latest** (official) | Maintained by Vercel; App Router + TS + Tailwind + ESLint defaults; includes `AGENTS.md` for coding agents | Minimal — we add content layer, analytics, AI proxy ourselves | **Selected** |
| T3 stack | Adds tRPC + Prisma | Over-engineered for no-DB Phase 1 | Rejected |
| Vite + React SPA | Lighter | Loses SSG/ISR, serverless API routes, SEO for resort pages | Rejected |

### Selected Starter: create-next-app (Official Next.js CLI)

**Rationale:** Matches PRD §7.1 exactly; production-ready defaults; Turbopack dev; App Router for SSG resort pages + Route Handlers for AI proxy. Verified via npm/docs: **create-next-app v16.2.x** (June 2026).

**Initialization Command (Epic 1 Story 1.1):**

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

> App lives in `web/` subdirectory to keep `content/`, `docs/`, and planning artifacts at repo root.

**Architectural Decisions Provided by Starter:**

| Area | Decision |
|------|----------|
| Language | TypeScript (strict) |
| Framework | Next.js 16.x App Router |
| Styling | Tailwind CSS v4 (bundled with latest template) |
| Linting | ESLint (Next.js config) |
| Structure | `src/app/` routes, `@/*` import alias |
| Dev server | Turbopack (default) |
| Agent guidance | `AGENTS.md` generated — extend with project rules |

**Note:** Project initialization is Epic 1 Story 1.1 — first implementation task.

---

## Core Architectural Decisions

### Decision Priority Analysis

**Critical (block implementation):**
- Next.js App Router + TypeScript + Tailwind
- Build-time markdown content loader (`gray-matter`)
- Serverless `/api/chat` with Anthropic proxy
- PostHog + consent banner
- Security headers + rate limiting

**Important (shape architecture):**
- Zustand for client UI state (quiz, onboarding, chat UI)
- Feature-based component organization
- Vercel deployment + env var strategy
- i18n scaffold (`next-intl` or lightweight custom — see below)

**Deferred (Phase 2+):**
- Supabase PostgreSQL + Auth
- Google Places API proxy
- Live snow search + shared cache
- Leaflet map integration (Phase 2 accommodation)
- CMS editorial workflow

### Data Architecture

**Phase 1 — no application database.**

| Data type | Storage | Access |
|-----------|---------|--------|
| Resort content | `content/resorts/*.md` | Build-time import via `src/lib/content/` |
| Site images | `content/site-images.md` | Build-time import |
| Onboarding complete | `localStorage` key `onboarding_complete` | Client only |
| Quiz answers (session) | React state / Zustand | Client only; not persisted Phase 1 |
| Analytics identity | PostHog anonymous ID | PostHog SDK |
| AI conversation | In-memory per session | Server route handler; no DB log Phase 1 |

**Content loader design:**

```
content/resorts/<slug>.md
    ↓ gray-matter parse
ResortFrontmatter (typed via Zod)
    ↓ filter content_status === 'published'
Resort[] index at build time
    ↓
getResortBySlug(slug) / getAllResorts()
```

**Validation:** Zod schema mirroring `CONTENT_MODEL.md` §4 + PRD §8.1. Dev warnings for unpublished slugs; build fails only if zero published resorts.

**Caching:**
- Resort pages: SSG with `revalidate` optional (default static at build)
- AI responses: no cache Phase 1 (stateless per request except in-memory session in route — use cookie or header for session id if needed)
- Phase 3 snow: edge cache TTL 6–24h (documented, not built)

### Authentication & Security

**Phase 1:** No user accounts. No JWT. No Supabase.

| Control | Implementation |
|---------|----------------|
| API keys | `ANTHROPIC_API_KEY`, `POSTHOG_KEY` etc. in Vercel env only — never `NEXT_PUBLIC_*` except PostHog project key |
| CSP | `next.config.ts` headers — `script-src 'self'`; whitelist PostHog/Plausible domains |
| HSTS | Vercel default + explicit header |
| Rate limiting | `@upstash/ratelimit` + Redis **or** in-memory limiter for MVP (document: upgrade before scale) |
| AI chat | 10 req/min/IP; message/token caps per §4.4 |
| Input sanitization | DOMPurify or strip HTML on chat input; never render raw user HTML |
| CORS | `/api/*` same-origin only |
| Dependencies | `npm audit` in CI; lockfile committed |

**Phase 2 auth (documented, not built):** Supabase Auth with HTTP-only cookies; JWT not in localStorage.

### API & Communication Patterns

**Pattern:** Next.js Route Handlers (REST-ish JSON).

| Route | Method | Purpose | Auth |
|-------|--------|---------|------|
| `/api/chat` | POST | AI resort Q&A proxy | Rate limit only |
| `/api/health` | GET | Smoke test | Public |

**Request/response (chat):**

```typescript
// POST /api/chat
// Request
{ "resortSlug": "niseko-united", "messages": [{ "role": "user"|"assistant", "content": string }], "sessionId": string }

// Response 200
{ "message": { "role": "assistant", "content": string }, "meta": { "model": string, "cached": boolean } }

// Response 429 — rate limit
{ "error": { "code": "RATE_LIMIT", "message": string } }

// Response 503 — kill-switch
{ "error": { "code": "AI_UNAVAILABLE", "message": string }, "fallback": { "suggestions": string[] } }
```

**Error standard:** `{ error: { code: string, message: string } }` — codes are `SCREAMING_SNAKE`.

**External integrations (Phase 1):**

| Service | Direction | Notes |
|---------|-----------|-------|
| Anthropic Messages API | Server outbound | Claude Sonnet primary; Haiku for routing (Phase 1 optional) |
| PostHog EU | Client + server | EU cloud region |
| Plausible | Client script | Privacy-clean pageviews |
| Unsplash | Build script only | `scripts/fetch_unsplash_assets.py`; not runtime Phase 1 |

### Frontend Architecture

| Decision | Choice | Rationale |
|----------|--------|-----------|
| State management | **Zustand** | Lightweight; quiz + onboarding + UI flags; PRD §7.1 allows Zustand or Context — pick one to avoid agent drift |
| Component org | **Feature folders** under `src/components/` | `resort/`, `quiz/`, `chat/`, `layout/`, `ui/` |
| Routing | App Router file-based | `src/app/page.tsx`, `explore/`, `info/`, `resorts/[slug]/`, `quiz/` |
| Styling | Tailwind + CSS variables | Map `DESIGN.md` tokens to `:root` in `globals.css` |
| Fonts | `next/font/google` | Cormorant Garamond + DM Sans |
| i18n | **`next-intl`** scaffold, EN only | Phase 2 TH/JP; strings in `messages/en.json` |
| Animations | CSS transitions + View Transitions API where supported | Per EXPERIENCE.md motion budget; no Framer Motion Phase 1 |
| Images | `next/image` | Remote patterns for Unsplash CDN; attribution component alongside |

**Quiz (Phase 1):** Scoring via `src/lib/quiz/score.ts` per [`docs/quiz-scoring.md`](../../docs/quiz-scoring.md). UI: `QuizModeToggle`, `ResortQuizCard`, **`QuizResultsReveal`**, **`QuizTopMatchCard`**, **`QuizRunnerUpRow`**, `getMatchReason()` — Serious header only; Cosmic interstitial ~800ms + header + subline; **podium top 3** on results (`features/quiz.md` § Results reveal + § Results podium). Shared enums for Serious/Cosmic; `quiz_completed` fires after reveal visible (`result_count: 3`).

### Infrastructure & Deployment

| Decision | Choice |
|----------|--------|
| Hosting | **Vercel** (production) |
| CI | GitHub Actions — lint, typecheck, build, `npm audit` |
| Environments | `development`, `preview`, `production` |
| Env vars | `.env.example` at repo root; `web/.env.local` for dev |

**Required environment variables:**

```bash
# web/.env.local (see .env.example)
ANTHROPIC_API_KEY=           # server only
AI_DAILY_KILL_SWITCH_USD=    # optional budget cap
NEXT_PUBLIC_POSTHOG_KEY=     # public project key
NEXT_PUBLIC_POSTHOG_HOST=    # EU host
NEXT_PUBLIC_PLAUSIBLE_DOMAIN=
NEXT_PUBLIC_APP_VERSION=1.0.0
UNSPLASH_ACCESS_KEY=         # scripts only, not web runtime Phase 1
```

**Monitoring:** Vercel analytics + PostHog for product events; no Sentry Phase 1 (add if error volume warrants).

---

## Implementation Patterns & Consistency Rules

### Critical Conflict Points

Areas where AI agents could diverge without these rules:

1. Resort slug format (`kebab-case` everywhere)
2. Analytics event names (must match `docs/tracking-plan.md` exactly)
3. API error shape
4. Component/file naming
5. Content type field names (match YAML frontmatter, not camelCase renames)
6. Import alias `@/*` only — no relative `../../` beyond sibling files

### Naming Patterns

**Content / data (match markdown frontmatter — snake_case):**

```
resort_slug: niseko-united
trail_map_url
content_status
last_reviewed
```

**TypeScript domain types (camelCase in TS, map at loader boundary):**

```typescript
// src/lib/content/types.ts
interface Resort {
  slug: string;           // from filename
  trailMapUrl: string;      // from trail_map_url
  contentStatus: 'draft' | 'published';
}
```

**API:** JSON fields `camelCase`; route params `[slug]` kebab-case.

**React components:** PascalCase files and exports — `ResortCard.tsx`, `FilterChip.tsx`.

**Routes:** lowercase kebab — `/resorts/[slug]`, `/getting-there` nested under resort.

**Analytics events:** exact `snake_case` from tracking plan — `resort_card_tapped`, not `resortCardTapped`.

**Zustand stores:** `useQuizStore`, `useOnboardingStore` — one store per domain.

### Structure Patterns

```
web/
├── src/
│   ├── app/                    # Routes only — thin pages
│   │   ├── layout.tsx          # Root layout, fonts, providers
│   │   ├── page.tsx            # Home
│   │   ├── explore/page.tsx
│   │   ├── info/page.tsx
│   │   ├── privacy/page.tsx
│   │   ├── terms/page.tsx
│   │   ├── quiz/page.tsx
│   │   ├── resorts/[slug]/page.tsx
│   │   ├── resorts/[slug]/getting-there/page.tsx
│   │   └── api/
│   │       ├── chat/route.ts
│   │       └── health/route.ts
│   ├── components/
│   │   ├── layout/             # BottomNav, TopBar, PageShell
│   │   ├── resort/             # ResortCard, QuickVerdict, etc.
│   │   ├── quiz/
│   │   ├── chat/
│   │   ├── onboarding/
│   │   └── ui/                 # BadgePill, SectionHeader, UnsplashAttribution
│   ├── lib/
│   │   ├── content/            # loader, types, zod schema
│   │   ├── quiz/               # score.ts, questions.ts
│   │   ├── ai/                 # prompts, retrieve, guardrails
│   │   ├── analytics/          # track.ts, posthog provider
│   │   └── utils/
│   ├── hooks/
│   ├── stores/                 # Zustand
│   └── styles/
│       └── tokens.css          # DESIGN.md CSS variables
├── messages/
│   └── en.json
├── public/
├── next.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

**Tests:** Co-located `*.test.ts` next to `lib/` modules; `*.test.tsx` next to complex components. No separate `tests/` tree Phase 1 except optional `e2e/` later.

### Format Patterns

**Analytics `track()` helper:**

```typescript
track('resort_card_tapped', {
  resort_slug: slug,        // always snake_case keys per tracking-plan
  list_context: 'home',
  consent_state: getConsentState(),
});
```

**Dates in UI:** ISO source → formatted with `Intl.DateTimeFormat`; season labels explicit: `"2025/26 season"`.

**External links:** always `target="_blank" rel="noopener noreferrer"`.

### Process Patterns

**Loading states:** `{ isLoading, error, data }` hook pattern or skeleton components per EXPERIENCE.md State Patterns.

**Errors:** User-facing copy warm and plain; technical detail in server logs only.

**Feature flags:** PostHog flags scaffold from Phase 1 for safe rollout (PRD §13).

### Enforcement Guidelines

**All AI agents MUST:**

- Read `architecture.md`, `DESIGN.md`, `EXPERIENCE.md`, and `docs/tracking-plan.md` before implementing a story
- Use `@/*` imports from `src/`
- Add analytics events to tracking plan in the same change as instrumentation
- Never commit secrets; never put Anthropic key in client bundle
- Match component names to UX spine (`ResortCard`, not `SkiResortCard`)

**Anti-patterns:**

- Fetching resort content client-side from markdown files
- Storing API keys in `NEXT_PUBLIC_*`
- Creating a database for Phase 1 features
- Inventing analytics event names not in tracking plan

---

## Project Structure & Boundaries

### Complete Project Directory Structure

```
japan_winter_sport/                    # repo root (existing)
├── content/                           # source of truth (existing)
│   ├── CONTENT_MODEL.md
│   ├── site-images.md
│   └── resorts/
│       ├── _TEMPLATE.md
│       └── *.md                       # 20 resort files
├── docs/
│   └── tracking-plan.md
├── scripts/
│   └── fetch_unsplash_assets.py
├── _bmad-output/                      # planning artifacts
├── japan_ski_prd.md
├── project-context.md
├── .env.example
└── web/                               # NEW — Next.js app (Epic 1 Story 1.1)
    ├── AGENTS.md                      # extend with project rules
    ├── README.md
    ├── package.json
    ├── next.config.ts
    ├── tailwind.config.ts
    ├── tsconfig.json
    ├── .env.local                     # gitignored
    ├── public/
    │   └── icons/
    ├── messages/en.json
    └── src/
        ├── app/                       # (see Structure Patterns)
        ├── components/
        ├── lib/
        ├── hooks/
        ├── stores/
        └── styles/tokens.css
```

### Architectural Boundaries

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser (client)                      │
│  Pages · Components · Zustand · PostHog/Plausible · localStorage │
└──────────────────────────┬──────────────────────────────────┘
                           │ same-origin fetch
┌──────────────────────────▼──────────────────────────────────┐
│                   Next.js (Vercel serverless)                  │
│  Route Handlers /api/chat  ·  RSC/SSG pages  ·  Middleware   │
└──────────┬─────────────────────────────┬────────────────────┘
           │ build-time read              │ HTTPS outbound
┌──────────▼──────────┐       ┌─────────▼─────────┐
│  content/*.md (git)  │       │  Anthropic API     │
└─────────────────────┘       │  PostHog (server)    │
                              └─────────────────────┘
```

**Boundary rules:**
- Client never calls Anthropic directly
- Client never parses markdown files — only receives typed props from RSC/build
- Content editors never touch `web/src/` — separate concern
- UX/visual changes reference `DESIGN.md`; behaviour references `EXPERIENCE.md`

### Requirements to Structure Mapping

| Epic | Primary locations |
|------|-------------------|
| Epic 1 Foundation | `web/src/app/layout.tsx`, `lib/content/`, `lib/analytics/`, `components/layout/`, `styles/tokens.css` |
| Epic 2 Discover | `app/page.tsx`, `app/explore/`, `app/quiz/`, `components/resort/`, `components/quiz/`, `lib/quiz/` |
| Epic 3 Detail | `app/resorts/[slug]/`, `components/resort/`, Getting There route |
| Epic 4 AI | `app/api/chat/`, `lib/ai/`, `components/chat/` |
| Epic 5 Info/Launch | `app/info/`, `app/privacy/`, `app/terms/`, satisfaction component |

### Data Flow

1. **Build:** `lib/content/loadResorts()` reads markdown → validates → generates static params for `/resorts/[slug]`
2. **Browse:** Home page receives `Resort[]` as props → client filters via Zustand/local state
3. **Quiz:** Answers in Zustand → `score.ts` ranks all 20 → **podium top 3** results view + Browse CTA → Explore
4. **Chat:** User message → POST `/api/chat` with `resortSlug` → server loads resort fields from in-memory index → Anthropic → JSON response
5. **Analytics:** Component events → `track()` → PostHog (consent-gated)

---

## Architecture Validation Results

### Coherence Validation ✅

**Decision compatibility:** Next.js SSG + serverless API + file-based content is coherent for Phase 1 scope. No database avoids auth complexity. Zustand + RSC props split is standard. Vercel hosts both static and functions.

**Pattern consistency:** snake_case analytics + camelCase TS + kebab slugs — mapping layer at content loader prevents drift.

**Structure alignment:** Feature folders map 1:1 to epics; content stays at repo root per PRD §11.7.

### Requirements Coverage Validation ✅

| FR | Supported by |
|----|--------------|
| FR-1 | Resort index + client filter + Home/Explore routes |
| FR-2 | SSG detail + UnsplashAttribution + trail map link |
| FR-3 | Getting There route + TransportRow from frontmatter |
| FR-4 | Quiz page + score.ts + Zustand |
| FR-5 | Onboarding overlay + localStorage |
| FR-6 | /api/chat + lib/ai guardrails |
| FR-7 | Info + legal static pages |

All 5 epics / 27 stories map to defined directories. NFR-3–9 addressed in Security and AI sections.

### Implementation Readiness Validation ✅

- Starter command documented with verified CLI version
- Env vars listed
- API contract sketched
- Naming and structure patterns explicit
- UX spines referenced for visual/behavioural contracts

### Gap Analysis

| Gap | Priority | Resolution |
|-----|----------|------------|
| Quiz scoring weights | Important | Implement in `lib/quiz/score.ts`; tune with Ninja persona test |
| Rate limiter backing store | Important | In-memory OK for launch; Upstash Redis before traffic spike |
| Shared-element transition | Nice-to-have | View Transitions API or CSS fallback; cosmetic per user note |
| AI session persistence | Important | Pass `sessionId` client-generated UUID; server tracks turn count in memory or edge KV |
| E2E test framework | Deferred | Add Playwright post-Epic 5 |

### Architecture Completeness Checklist

**Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**Implementation Patterns**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High for Phase 1 foundation; medium for quiz scoring nuance (tune during Epic 2).

**Key Strengths:**
- File-based content decouples editorial from app releases
- Clear server/client boundary for AI and secrets
- Strong alignment with approved PRD, UX spines, and epics
- Explicit agent consistency rules

**Areas for Future Enhancement:**
- Supabase auth + Plan tab (Phase 3)
- Upstash rate limiting + snow cache
- Leaflet map module (Phase 2)
- CMS export pipeline (Phase 3)

### Implementation Handoff

**AI Agent Guidelines:**

1. Implement stories in epic order (Epics 1 → 5)
2. Epic 1 Story 1.1 = run `create-next-app` command above into `web/`
3. Wire design tokens from `DESIGN.md` before feature UI
4. Instrument analytics per `docs/tracking-plan.md` in same PR as feature
5. Never wire Contentful/Sanity (PRD §11.7)

**First Implementation Priority:**

```bash
npx create-next-app@latest web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm --yes
```

Then: design tokens (`tokens.css`) → content loader → layout shell with BottomNav.

**Suggested next BMAD steps:**
1. **[IR] Check Implementation Readiness** — `bmad-check-implementation-readiness`
2. **[SP] Sprint Planning** — `bmad-sprint-planning`
3. **[DS] Dev Story** — Epic 1 Story 1.1

---

*Architecture workflow complete — 2026-06-12*
