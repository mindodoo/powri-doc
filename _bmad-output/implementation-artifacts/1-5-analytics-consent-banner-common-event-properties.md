# Story 1.5: Analytics, Consent Banner & Common Event Properties

Status: done

baseline_commit: NO_VCS

<!-- Validation optional: bmad-create-story:validate (VS) before bmad-dev-story -->

## Story

As a **product owner**,
I want privacy-first analytics with consent gating from day one,
So that we can measure funnels without violating GDPR/PDPA.

## Acceptance Criteria

1. **Given** a first-time visitor with no consent stored  
   **When** the app loads  
   **Then** PostHog (EU) and Plausible initialize in anonymous/cookieless mode (NFR-4, NFR-8)

2. **Given** no prior consent choice  
   **When** the app renders  
   **Then** a consent banner gates identified/persistent tracking until accepted (deny stays cookieless)

3. **Given** the tracking plan §1  
   **When** `track()` is called  
   **Then** common properties are attached: `locale`, `device_type`, `is_logged_in`, `consent_state`, `app_version`

4. **Given** route navigation  
   **When** pages change  
   **Then** `$pageview` is captured in PostHog; Plausible receives pageviews when configured

5. **Given** any analytics payload  
   **When** events are sent  
   **Then** no PII is included in properties

## Tasks / Subtasks

- [x] **Dependencies & env** (AC: 1)
  - [x] `npm install posthog-js` in `web/`
  - [x] Document env vars in `web/.env.example`: `NEXT_PUBLIC_POSTHOG_KEY`, `NEXT_PUBLIC_POSTHOG_HOST`, `NEXT_PUBLIC_PLAUSIBLE_DOMAIN`, `NEXT_PUBLIC_APP_VERSION`
  - [x] Graceful no-op when keys missing (dev/build without credentials)

- [x] **Consent module** (AC: 2)
  - [x] `web/src/lib/analytics/consent.ts` — `getConsentState()`, `setConsentState()`, localStorage key `analytics_consent` (`anonymous` | `granted` | `denied`)
  - [x] Default before choice: treat as `anonymous`, show banner

- [x] **Analytics core** (AC: 3, 5)
  - [x] `web/src/lib/analytics/device.ts` — `getDeviceType()` mobile/tablet/desktop
  - [x] `web/src/lib/analytics/commonProperties.ts` — common props per tracking-plan §1
  - [x] `web/src/lib/analytics/track.ts` — `track(event, props)` merges common props, strips unsafe keys
  - [x] `web/src/lib/analytics/posthog.ts` — init/reconfigure based on consent (memory vs persistence)
  - [x] `web/src/lib/analytics/index.ts` — public exports

- [x] **Client providers** (AC: 1, 2, 4)
  - [x] `web/src/components/analytics/AnalyticsProvider.tsx` — PostHog init, Plausible script, pageview on route change
  - [x] `web/src/components/analytics/ConsentBanner.tsx` — Accept / Decline; Theme A styling; fixed above BottomNav
  - [x] Wire `AnalyticsProvider` in `layout.tsx`; `ConsentBanner` in `AppShell`

- [x] **Validate** (AC: 1–5)
  - [x] `npm run build` + `npm run lint` pass
  - [x] Without env keys: app loads, no runtime errors
  - [x] Export `track` ready for Epic 2+ feature instrumentation

- [x] **Out of scope**
  - Feature-specific events (Epic 2+)
  - CSP whitelist for PostHog/Plausible domains (Story 1.6)
  - i18n locale detection (Story 1.7 — hardcode `en` for now)
  - Session replay, feature flags UI

## Dev Notes

### PostHog init pattern (consent-aware)

```typescript
// anonymous / denied → memory persistence, cookieless
posthog.init(key, {
  api_host: 'https://eu.i.posthog.com',
  persistence: 'memory',
  person_profiles: 'identified_only',
  capture_pageview: false,
});

// granted → upgrade persistence
posthog.set_config({ persistence: 'localStorage+cookie' });
```

Skip init entirely if `NEXT_PUBLIC_POSTHOG_KEY` is unset.

### Plausible

Load via `next/script` when `NEXT_PUBLIC_PLAUSIBLE_DOMAIN` is set. Fire manual `pageview` on App Router navigation (Plausible does not auto-track SPA routes).

### Consent banner UX

- Fixed bottom, **above** BottomNav (`bottom-14` + safe area)
- z-index above nav (`z-[60]`)
- Copy: short GDPR/PDPA notice; Accept / Decline buttons
- Accept → `granted`; Decline → `denied` (stay cookieless, no persistent tracking)

### track() signature

```typescript
track('resort_card_tapped', {
  resort_slug: 'niseko-united',
  list_context: 'home',
});
// Auto-adds: locale, device_type, is_logged_in, consent_state, app_version
```

### PII guardrails

Never pass: email, name, IP, raw search queries (unless consent=granted — not used in Story 1.5). Block keys matching `/email|name|phone|ip/i` in track sanitizer.

### Previous story intelligence

- AppShell + BottomNav at `z-50`, height 56px — banner stacks above
- Layout is server component — analytics must be client providers
- `locale: 'en'` until Story 1.7

### References

- [Source: epics.md § Story 1.5](../planning-artifacts/epics.md)
- [Source: docs/tracking-plan.md §0–1](../../docs/tracking-plan.md)
- [Source: architecture.md § Data Architecture / Analytics](../planning-artifacts/architecture.md)
- [Source: japan_ski_prd.md §13](../../japan_ski_prd.md)

## Dev Agent Record

### Agent Model Used

Composer

### Debug Log References

- `npm run build` — pass
- `npm run lint` — pass (useSyncExternalStore for consent hydration)

### Completion Notes List

- Added `posthog-js`; consent-aware init (memory/cookieless pre-grant, persistence on accept)
- Plausible script + manual SPA pageviews when domain configured
- `track()` helper with common properties + PII key sanitizer
- Consent banner above BottomNav; Accept/Decline stored in localStorage

### File List

- `web/package.json` (modified — posthog-js)
- `web/package-lock.json` (modified)
- `web/.env.example` (new)
- `web/src/lib/analytics/consent.ts` (new)
- `web/src/lib/analytics/device.ts` (new)
- `web/src/lib/analytics/commonProperties.ts` (new)
- `web/src/lib/analytics/posthog.ts` (new)
- `web/src/lib/analytics/track.ts` (new)
- `web/src/lib/analytics/index.ts` (new)
- `web/src/types/plausible.d.ts` (new)
- `web/src/components/analytics/AnalyticsProvider.tsx` (new)
- `web/src/components/analytics/ConsentBanner.tsx` (new)
- `web/src/app/layout.tsx` (modified)
- `web/src/components/layout/AppShell.tsx` (modified)

## Senior Developer Review (AI)

**Review date:** 2026-06-12  
**Outcome:** Approve

### Summary

All five ACs satisfied. Consent-aware PostHog, Plausible pageviews, `track()` with common properties, banner UX above BottomNav.

### Review Findings

- [x] [Review][Defer] Plausible CDN script missing SRI — addressed in Story 1.6
- [x] [Review][Defer] CSP whitelist for PostHog/Plausible domains — Story 1.6
