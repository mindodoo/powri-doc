# Story 1.7: i18n Scaffold (English Only)

Status: done

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **developer**,
I want i18n infrastructure ready for Phase 2 languages,
So that EN ships now without rework later.

## Acceptance Criteria

1. **Given** Phase 1 English-only scope (NFR-14)  
   **When** the app renders  
   **Then** all user-facing UI strings route through an i18n layer defaulting to `en`

2. **Given** analytics common properties  
   **When** events fire  
   **Then** `locale` defaults to `en` (from active locale)

3. **Given** Phase 1 scope  
   **When** messages are inspected  
   **Then** only `messages/en.json` exists — no TH/JP copy in UI

## Tasks / Subtasks

- [x] **Install & configure next-intl** (AC: 1, 3)
  - [x] `npm install next-intl` in `web/`
  - [x] `web/src/i18n/routing.ts` — locales: `['en']`, `localePrefix: 'never'`
  - [x] `web/src/i18n/request.ts` — load `messages/en.json`
  - [x] Wrap `next.config.ts` with `createNextIntlPlugin`

- [x] **Messages file** (AC: 1, 3)
  - [x] Create `web/messages/en.json` with all Phase 1 UI strings (nav, consent, pages, metadata)
  - [x] No `th.json` / `ja.json`

- [x] **Wire providers** (AC: 1)
  - [x] Update `layout.tsx` — `NextIntlClientProvider`, `getMessages()`, `lang={locale}`
  - [x] Merge next-intl middleware with existing API middleware in `middleware.ts`

- [x] **Migrate UI strings** (AC: 1)
  - [x] `BottomNav`, `ConsentBanner` — `useTranslations`
  - [x] `page.tsx`, `explore/page.tsx`, `info/page.tsx`, `resorts/[slug]/page.tsx` — `getTranslations`
  - [x] Resort content (markdown) stays as-is — not i18n'd in this story

- [x] **Analytics locale** (AC: 2)
  - [x] Update `commonProperties.ts` to read locale from `document.documentElement.lang` (fallback `en`)

- [x] **Validate** (AC: 1–3)
  - [x] `npm run build` + `npm run lint` pass
  - [x] `<html lang="en">` preserved via `getLocale()`

- [x] **Out of scope**
  - TH/JP message files (Phase 2)
  - Locale switcher UI
  - Translating resort markdown content

## Dev Notes

### Architecture choice

Use **`next-intl`** per architecture.md. Phase 1: `localePrefix: 'never'` — no `/en` URL prefix.

### Middleware merge pattern

```typescript
export default function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/api/')) {
    return handleApiMiddleware(request);
  }
  return intlMiddleware(request);
}
```

### Message key structure

```json
{
  "metadata": { "title": "...", "description": "..." },
  "nav": { "home": "Home", "explore": "Explore", "info": "Info" },
  "consent": { ... },
  "home": { ... },
  "explore": { ... },
  "info": { ... },
  "resort": { ... }
}
```

### Previous story intelligence

- Story 1.5: `locale: 'en'` hardcoded in analytics — replace with dynamic read
- Story 1.6: middleware handles `/api/*` — extend, don't replace
- All pages are server components except BottomNav + ConsentBanner

### References

- [Source: epics.md § Story 1.7](../planning-artifacts/epics.md)
- [Source: architecture.md § i18n](../planning-artifacts/architecture.md)
- [Source: japan_ski_prd.md NFR-14](../../japan_ski_prd.md)

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- Installed `next-intl@^4.13.0`; configured routing with single `en` locale and `localePrefix: 'never'`
- Created `messages/en.json` with metadata, nav, consent, home, explore, info, resort keys
- Root layout uses `NextIntlClientProvider`, dynamic `lang={locale}`, and `generateMetadata()` via `getTranslations`
- Merged intl middleware with API CORS/rate-limit middleware; broadened matcher for page routes
- Migrated all Phase 1 UI strings; resort markdown content unchanged
- Fixed routing: moved pages under `app/[locale]/` (required by next-intl middleware even with `localePrefix: 'never'`)
- Analytics `locale` reads from `document.documentElement.lang` (client-only, fallback `en`)
- Fixed `corsForbiddenResponse()` to return `NextResponse` for middleware type compatibility
- Build + lint pass; 29 static pages generated

### File List

- `web/package.json` (modified — next-intl dependency)
- `web/package-lock.json` (modified)
- `web/messages/en.json` (new)
- `web/src/i18n/routing.ts` (new)
- `web/src/i18n/request.ts` (new)
- `web/next.config.ts` (modified — createNextIntlPlugin)
- `web/src/middleware.ts` (modified — merged intl + API)
- `web/src/app/layout.tsx` (modified — NextIntlClientProvider, i18n metadata)
- `web/src/app/[locale]/layout.tsx` (new)
- `web/src/app/[locale]/page.tsx` (new — moved from app root)
- `web/src/app/[locale]/explore/page.tsx` (new)
- `web/src/app/[locale]/info/page.tsx` (new)
- `web/src/app/[locale]/resorts/[slug]/page.tsx` (new)
- `web/src/components/layout/BottomNav.tsx` (modified — useTranslations)
- `web/src/components/analytics/ConsentBanner.tsx` (modified — useTranslations)
- `web/src/lib/analytics/commonProperties.ts` (modified — dynamic locale)
- `web/src/lib/api/cors.ts` (modified — NextResponse return type)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Review Findings

- [x] [Review][Defer] Locale switcher and TH/JP files deferred to Phase 2 per NFR-14
- [x] [Review][Defer] Resort markdown content not i18n'd — correct for this story scope
