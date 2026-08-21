# Deferred Work

## Deferred from: PO decision — Story 12.3 flat comment thread (2026-08-13)

- **Story 12.3** (flat comment thread under Experience section) — deferred; PO no longer wants comment-on-resort UX in Phase 2
- Replacement direction: connect snow riders through another mechanism (**TBD** — PO researching best-fit community feature; not scoped yet)
- **Do not implement:** `CommentList`, `/api/comments` routes, "Add a comment" CTA, `comment_submitted` analytics, comment report/hide API until replacement feature is defined
- **Story 12.4 (confirmed):** reviews-only report + admin hide — no comment API

## Deferred from: code review of 12-4-report-content-admin-hide (2026-08-13)

- Guest Report opens SignInSheet (AC6) but does not resume into `ReportReviewDialog` after auth — uses `openSignIn` rather than `requireAuthForAction` (`web/src/components/experience/ReviewCard.tsx`)
- `GET /api/reviews?admin=true` 401/403 is treated as a generic list load failure, blanking the review section for an expired admin session instead of falling back to the published-only fetch (`web/src/components/experience/ReviewList.tsx`)
- Hiding a review does not resolve/dismiss matching `content_reports` rows — no report-resolution UI in 12.4 (`web/src/app/api/admin/reviews/[id]/hide/route.ts`)

## Deferred from: code review of 1-1-initialize-next-js-project-deployment-baseline (2026-06-12)

- Nested `.git` inside `web/` from create-next-app — remove when repo root is initialized
- Starter `page.tsx` Geist/zinc/dark classes — addressed in Story 1.2

## Deferred from: code review of 10-4-saved-list-page-login-merge (2026-07-10)

- `saved_count` analytics may exceed visible cards when stale slugs in provider (`SavedPageContent.tsx:29`)
- Non-atomic merge count gate TOCTOU on concurrent tab login (`merge/route.ts:46`)
- Orphaned localStorage on session-restore without merge when `previousAuthenticated` is null (`SavedResortsProvider.tsx:168`)

## Deferred from: code review of 12-2-1-review-photo-storage-r2 (2026-08-11)

- Orphaned unconfirmed uploads are never cleaned up in either storage provider — presigned PUT with no confirm has no lifecycle/expiry rule; pre-existing pattern carried over from Story 12.2, not introduced by this story (`web/src/app/api/reviews/[id]/photos/route.ts`)
- CSP's R2 `connect-src` wildcard assumes virtual-hosted-style S3 addressing without explicitly pinning `forcePathStyle: false` on the `S3Client` — correct today per AC16 but fragile if AWS SDK addressing defaults change (`web/src/lib/security/csp.ts`, `web/src/lib/reviews/storage/r2Storage.ts`)

## Deferred from: code review of 12-2-1-review-photo-storage-r2, round 2 (2026-08-13)

- `deleteReviewPhotoRows` silently returns `{ storageRefs: [] }` and skips deletion when the initial `select` query errors — pre-existing silent-failure mode carried over unchanged from Story 12.2, not introduced by this story (`web/src/lib/reviews/mutations.ts`)
- `isNotFoundError` treats any HTTP 404 status as "not found" (via the `$metadata.httpStatusCode === 404` fallback), which could also mask a genuinely wrong bucket/endpoint (e.g. `NoSuchBucket` also returns 404) as a benign "not yet migrated" state (`web/src/lib/reviews/storage/r2Storage.ts:25-33`)
- `SupabaseReviewPhotoStorage.createUploadUrl` ignores its `contentType` parameter while `R2ReviewPhotoStorage` pins `ContentType` on the presigned PUT — the shared `ReviewPhotoStorage` interface's content-type enforcement at upload time silently differs by provider; low risk since the final compressed content-type is always set correctly at `overwrite()` (`web/src/lib/reviews/storage/supabaseStorage.ts`)
- Full enforcement of a max-upload-size on the R2 bucket needs an R2 bucket-level policy or a presigned-POST redesign with size-range conditions — infra/PO decision, not a pure code change (`web/src/lib/reviews/storage/r2Storage.ts:99-113`)

## Deferred from: code review of ux-5-sign-out-no-signin-flash (2026-08-21)

- `auth_sign_out` is listed in the tracking plan / `phase2Events.ts` but has never been instrumented (`instrumentationFiles: []`); AC5 only requires it to keep firing if it already did — pre-existing analytics gap, not introduced by UX-5 (`web/src/lib/analytics/phase2Events.ts:33-35`)

## Deferred from: code review of ux-7-account-remove-back (2026-08-21)

- UX-7 e2e could click hamburger on `/account` and assert `app-menu-sheet` opens — mirrors Saved test; visibility + no-Back guard is sufficient for merge (`web/e2e/smoke.spec.ts:298-308`)

## Deferred from: code review of ux-9-mobile-explore-search-icon (2026-08-21)

- UX-9 e2e does not assert `search_focused` with `surface=explore` — optional; `ResortSearchField` receives `listContext="explore"` by inspection (`web/src/components/layout/DiscoveryTopBar.tsx:128`)
- Explore search-panel close (X toggle) not e2e-automated — mirrors Home; open + results assertion sufficient for merge (`web/e2e/smoke.spec.ts:312-327`)
