# Deferred Work

## Deferred from: code review of 1-1-initialize-next-js-project-deployment-baseline (2026-06-12)

- Nested `.git` inside `web/` from create-next-app — remove when repo root is initialized
- Starter `page.tsx` Geist/zinc/dark classes — addressed in Story 1.2

## Deferred from: code review of 10-4-saved-list-page-login-merge (2026-07-10)

- `saved_count` analytics may exceed visible cards when stale slugs in provider (`SavedPageContent.tsx:29`)
- Non-atomic merge count gate TOCTOU on concurrent tab login (`merge/route.ts:46`)
- Orphaned localStorage on session-restore without merge when `previousAuthenticated` is null (`SavedResortsProvider.tsx:168`)
