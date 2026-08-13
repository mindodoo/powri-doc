---
baseline_commit: 369d1b381fb1edd3dad2fea7fe7fdc19eab1c950
---

# Story 12.2.1: Review Photo Storage — R2 Migration + Compression

Status: done

<!-- Ultimate context engine analysis completed - comprehensive developer guide created -->

## Story

As a **platform operator**,  
I want review photos stored on scalable external object storage with smaller file sizes,  
So that Supabase’s 1 GB free file quota is not exhausted quickly as UGC grows.

**Source shard(s):** [`epics/phase2/shards/epic-12-community-reviews.md`](../../planning-artifacts/epics/phase2/shards/epic-12-community-reviews.md) · [`prds/phase2/shards/06-features-reviews-comments.md`](../../planning-artifacts/prds/phase2/shards/06-features-reviews-comments.md) · architecture [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §3.4 · PO infra research 2026-08-11 — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **12.2** (signed URL upload flow, `review_photos` table, `photoResize.ts`, `backgroundPhotoUpload.ts`, owner kebab edit/delete).

**Blocks:** Story **12.4** (Report + admin hide) — **12.3** (flat comment thread) is **deferred** until after 12.2.1 and 12.4 per PO 2026-08-11.

**Epic note:** Story **15.3** gates public UGC launch legally — this story reduces infra risk before that gate. Passport photos and avatars remain out of scope.

---

## PO decisions (confirmed 2026-08-11)

| Topic | Decision |
|-------|----------|
| **Primary provider** | **Cloudflare R2** for new review photo blobs |
| **Compression (Phase 0)** | Always output **WebP** on confirm; max long edge **1600px**, quality **~80**; stored max **2 MB** (client input cap stays **20 MB**) |
| **Path extension** | Always **`.webp`** in `storage_path` (issued at upload URL step; confirm overwrites with WebP bytes) |
| **Supabase role** | Keep Postgres metadata (`review_photos`); stop new blob writes to Supabase `review-photos` bucket after cutover |
| **URL strategy** | Keep `storage_path` column; add `storage_provider text not null default 'supabase'` (`supabase` \| `r2`) for dual-read during migration |
| **Public access** | R2 public bucket via `*.r2.dev` initially; custom domain (`photos.powri.app`) deferred |
| **Feature flag** | Env `REVIEW_PHOTO_STORAGE_PROVIDER=supabase\|r2` — default `r2` in production after cutover; `supabase` when R2 creds absent (local dev) |
| **Existing photos** | One-time migration script in **same PR** copies Supabase objects → R2, updates rows; dual-read URL builder until verified |
| **Migration safety** | Script is idempotent; do **not** delete Supabase source objects in this story — PO deletes manually after sign-off |
| **Rollback** | Flag back to `supabase` for new uploads only — no auto-revert of migrated blobs |
| **Sprint order** | **12.2.1 before 12.4**; **12.3 deferred** |
| **Analytics** | No new events — existing `review_submitted` / `review_edited` `photo_count` unchanged |

---

## Acceptance Criteria

### A. Phase 0 — Compression (ships first, no infra change)

1. **Given** a photo confirm request  
   **When** the server processes bytes (any input ≤ 20 MB)  
   **Then** stored output is **WebP**, max long edge **1600px**, quality **~80**, and ≤ **2 MB**

2. **And** photos already ≤ 2 MB are still re-encoded to WebP for consistency

3. **And** if processing cannot produce ≤ 2 MB, confirm returns **400** with a clear message (same UX pattern as Story 12.2)

4. **And** `MAX_REVIEW_PHOTO_BYTES` updated to **2 MB**; `MAX_REVIEW_PHOTO_INPUT_BYTES` remains **20 MB**

5. **And** `POST /api/reviews/[id]/photos` issues `storagePath` ending in **`.webp`** regardless of client-declared MIME

6. **And** unit tests in `photoResize.test.ts` cover WebP output, dimension cap, and reject-over-limit

### B. Phase 1 — Storage abstraction

7. **Given** the review photo storage layer  
   **When** imported by API routes and mutations  
   **Then** a single adapter interface exposes: `createUploadUrl`, `download`, `overwrite`, `delete`, `publicUrl`

8. **And** `SupabaseReviewPhotoStorage` preserves current 12.2 behavior when provider = `supabase`

9. **And** `R2ReviewPhotoStorage` implements the same interface using S3-compatible presigned PUT via `@aws-sdk/client-s3` + `@aws-sdk/s3-request-presigner`

10. **And** factory reads `REVIEW_PHOTO_STORAGE_PROVIDER` and required R2 env vars (see Dev Notes)

### C. Phase 2 — R2 for new uploads

11. **Given** `REVIEW_PHOTO_STORAGE_PROVIDER=r2` and valid credentials  
    **When** owner calls `POST /api/reviews/[id]/photos`  
    **Then** response includes presigned **PUT** URL for R2 (same response shape: `{ photoId, storagePath, uploadUrl, token? }`)

12. **Given** client upload completes  
    **When** confirm runs  
    **Then** server downloads from R2, applies compression (AC A), overwrites object with WebP bytes + `image/webp` content type, inserts `review_photos` row with `storage_provider = 'r2'`

13. **Given** owner deletes/edits photos or soft-deletes review  
    **Then** R2 objects are deleted via adapter (best-effort, same as today)

14. **And** `buildReviewPhotoPublicUrl` resolves URL by `storage_provider` — Supabase host for legacy rows, `R2_PUBLIC_BASE_URL` for R2 rows

15. **And** `next.config.ts` `images.remotePatterns` includes R2 public hostname (when env set)

16. **And** CSP already allows `https:` for images — no CSP change required (`web/src/lib/security/csp.ts` `img-src … https:`)
    **Correction (found in manual QA, 2026-08-11):** `img-src` covers `<img>` rendering only. The client-side presigned **PUT** (`fetch` from `reviewApiClient.ts`) is governed by `connect-src`, which did **not** include the R2 upload host — the browser silently blocked the PUT (no network entry, no console error surfaced by default). Fixed by adding a dynamic `https://*.{R2_ACCOUNT_ID}.r2.cloudflarestorage.com` origin to `connect-src` in `csp.ts`.

### D. Phase 3 — Migration & dual-read (same PR)

17. **Given** existing rows with `storage_provider = 'supabase'` (or null treated as supabase)  
    **When** public API returns review photos  
    **Then** URLs still resolve correctly (no broken images)

18. **Given** migration script `scripts/migrate-review-photos-to-r2.ts` (or equivalent)  
    **When** run against staging/production with R2 creds  
    **Then** each Supabase object is copied to R2 at the same key, row updated to `storage_provider = 'r2'`, idempotent on re-run (skip if R2 object exists)

19. **And** script logs success/failure counts per row; failures do **not** delete source Supabase objects

20. **And** after migration + provider flag `r2`, new uploads never write to Supabase `review-photos` bucket

### E. Non-regression

21. **And** background upload queue (`backgroundPhotoUpload.ts`) works unchanged from the client’s perspective

22. **And** photo grid, lightbox, and account export still receive valid public URLs

23. **And** rate limits (`upload` 10/min/user) unchanged

24. **And** `npm run lint && npm run build && npm run test:unit` pass

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `photoResize.test.ts` (WebP, 2 MB cap, 1600px); storage adapter tests with mocked S3/Supabase clients; `photoUrl.test.ts` dual-read cases
- [x] **API route tests:** photo confirm resize path; signed URL issuance for R2 provider (mocked `@aws-sdk`)
- [x] **Analytics:** N/A — no new events
- [x] **Content / resorts:** `npm run test:launch` — no resort content changes expected; run if build/env touched
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **Manual QA (PO):** upload 3 photos on staging with R2; verify grid + lightbox; edit remove/add; delete review cleans R2; run migration script on staging with existing Supabase photos; legacy URLs work pre-migration, R2 URLs post-migration
- [x] **Env docs:** R2 vars in `web/.env.example` + [`docs/qa/phase2/deploy-environment.md`](../../../docs/qa/phase2/deploy-environment.md)

---

## Tasks / Subtasks

- [x] DB migration `008_review_photos_storage_provider.sql` — add `storage_provider text not null default 'supabase'` + check constraint (AC: 12, 17)
- [x] Phase 0: update `photoResize.ts`, `validation.ts`, `photos/route.ts` (`.webp` path), confirm route; extend tests (AC: 1–6)
- [x] Phase 1: `web/src/lib/reviews/storage/` — `types.ts`, `supabaseStorage.ts`, `r2Storage.ts`, `index.ts` factory (AC: 7–10)
- [x] Phase 2: wire `photos/route.ts`, `photos/confirm/route.ts`, `mutations.ts`, `queries.ts` + `photoUrl.ts` through adapter (AC: 11–16)
- [x] Phase 3: `scripts/migrate-review-photos-to-r2.ts` + npm script entry (AC: 18–20)
- [x] Config: `next.config.ts` R2 hostname helper, `web/.env.example`, deploy-environment.md (AC: 15)
- [x] Verification commands (Testing section)

### Review Findings

- [x] [Review][Patch] Pin storage provider at upload time instead of re-resolving it — `POST /photos` echoes `storageProvider` in its response; the client sends it back on `POST /photos/confirm` (and on reject/cleanup), and confirm/cleanup use that pinned value instead of re-calling `getActiveReviewPhotoStorage()`. Prevents confirm/cleanup from silently targeting the wrong backend if env config changes between the two requests. Resolved 2026-08-11: PO chose "echo provider in response, client sends it back" over path-embedding or a server-side map. [`web/src/app/api/reviews/[id]/photos/route.ts`, `web/src/app/api/reviews/[id]/photos/confirm/route.ts`, `web/src/lib/reviews/mutations.ts`, `web/src/lib/reviews/reviewApiClient.ts`]
- [x] [Review][Patch] Fire a PostHog event when storage falls back to Supabase due to missing/misconfigured R2 config, in addition to the existing `console.warn` — makes silent production fallback visible in analytics rather than only in logs. Resolved 2026-08-11: PO chose to add a PostHog event over accepting warn-only or fully deferring. [`web/src/lib/reviews/storage/env.ts:59`]
- [x] [Review][Patch] `photoUrl.ts` bypasses the validated R2 base-URL getter — `buildR2ReviewPhotoPublicUrl()` reads `process.env.R2_PUBLIC_BASE_URL` directly instead of reusing `getR2PublicBaseUrl()` from `storage/env.ts`, so the guard against accidentally configuring the private S3 endpoint (`*.r2.cloudflarestorage.com`) never protects the actual `<img>`-rendering path. `getR2PublicBaseUrl()` has no server-only imports, so it's safe to reuse from client-safe `photoUrl.ts`. [`web/src/lib/reviews/photoUrl.ts:13`]
- [x] [Review][Patch] `buildReviewPhotoPublicUrl` throws for r2 rows with no base URL configured — one misconfigured/legacy row throws inside `mapReviewRow`'s `.map()`, failing the entire review list response instead of degrading gracefully for just that photo. [`web/src/lib/reviews/queries.ts:46`, `web/src/lib/reviews/photoUrl.ts:13`]
- [x] [Review][Patch] Blanket `catch` in existence/download checks masks real infra failures as "not found" — `R2ReviewPhotoStorage.objectExists`/`download` and the migration script's `objectExists` (`HeadObjectCommand`) treat auth failures, network errors, and throttling identically to a genuine 404, so a misconfigured credential during migration looks like "not yet migrated" rather than a hard failure. [`web/src/lib/reviews/storage/r2Storage.ts:88`, `web/scripts/migrate-review-photos-to-r2.ts:79`]
- [x] [Review][Patch] Migration script has no pagination — `select('id, storage_path, storage_provider').eq('storage_provider', 'supabase')` relies on Supabase's default row cap; legacy rows beyond that limit are silently skipped and the run still reports success. [`web/scripts/migrate-review-photos-to-r2.ts:100`]
- [x] [Review][Patch] Migration script uses `import.meta.dirname` with no fallback — unsupported before Node 20.11/21.2; add a `fileURLToPath(import.meta.url)` fallback in case the deploy/CI runtime is ever pinned older. [`web/scripts/migrate-review-photos-to-r2.ts:17`]
- [x] [Review][Patch] `next.config.ts` R2 remotePattern hardcodes `protocol: 'https'` — if `R2_PUBLIC_BASE_URL` were ever `http://`, `next/image` would block all R2 photos since the pattern wouldn't match. Derive protocol from the parsed URL instead. [`web/next.config.ts:10`]
- [x] [Review][Patch] `reviewApiClient.ts` types the issue-response `token` as required (`token: string`) but the R2 adapter's `createUploadUrl` never returns one — type is inaccurate for the R2 path; change to `token?: string`. [`web/src/lib/reviews/reviewApiClient.ts:114`]
- [x] [Review][Patch] Migration script hand-rolls its own `S3Client` (duplicating the checksum-calculation workaround) instead of reusing `R2ReviewPhotoStorage`'s client construction — the two can drift on the next AWS SDK bump. [`web/scripts/migrate-review-photos-to-r2.ts:44`, `web/src/lib/reviews/storage/r2Storage.ts:16`]
- [x] [Review][Patch] Bucket name `'review-photos'` hardcoded in three places with no shared constant (migration script, `supabaseStorage.ts`) — extract to one exported constant. [`web/scripts/migrate-review-photos-to-r2.ts:12`, `web/src/lib/reviews/storage/supabaseStorage.ts:6`]
- [x] [Review][Patch] Best-effort deletes swallow errors with zero logging in both storage adapters — failed deletes during photo edit/removal leave orphaned blobs with no visibility, which matters more during an active dual-provider migration. [`web/src/lib/reviews/storage/r2Storage.ts:139`, `web/src/lib/reviews/storage/supabaseStorage.ts:59`]
- [x] [Review][Patch] `R2ReviewPhotoStorage`'s core methods (`download`, `overwrite`, `delete`, `streamToUint8Array` branching) and the migration script have no/thin unit coverage — `r2Storage.test.ts` only covers `createUploadUrl` and `publicUrl`, despite the DoD checklist marking storage-adapter tests as done. [`web/src/lib/reviews/storage/r2Storage.test.ts`, `web/scripts/migrate-review-photos-to-r2.ts`]
- [x] [Review][Patch] `env.test.ts` has no happy-path assertion — every test covers a fallback/warning scenario; none confirms `resolveActiveReviewPhotoStorageProvider()` returns `'r2'` when all credentials are present. [`web/src/lib/reviews/storage/env.test.ts`]
- [x] [Review][Patch] `.env.example` defaults `REVIEW_PHOTO_STORAGE_PROVIDER=r2` with blank R2 credentials — guarantees the "R2 intended but misconfigured" warning on every fresh local checkout that copies the example as-is. [`web/.env.example:31`]
- [x] [Review][Patch] CORS setup doc ships a literal placeholder `"https://your-production-domain"` in a copy-pasteable JSON block with no callout to replace it. [`docs/qa/phase2/deploy-environment.md:55`]
- [x] [Review][Defer] Orphaned unconfirmed uploads are never cleaned up in either provider [`web/src/app/api/reviews/[id]/photos/route.ts`] — deferred, pre-existing pattern carried over from Story 12.2 (presigned PUT with no confirm has never had a lifecycle/expiry rule); out of scope for this storage-migration story.
- [x] [Review][Defer] CSP's R2 `connect-src` wildcard assumes virtual-hosted-style S3 addressing without explicitly pinning `forcePathStyle: false` on the `S3Client` [`web/src/lib/security/csp.ts:34`, `web/src/lib/reviews/storage/r2Storage.ts:16`] — deferred, correct today per AC16 but a forward-looking robustness concern if AWS SDK addressing defaults ever change.

### Review Findings (round 2, post manual-QA — 2026-08-13)

- [x] [Review][Patch] Confirm route has no `try`/`catch` around `downloadReviewPhotoBytes`/`overwriteReviewPhotoBytes` — a non-404 R2 error (auth failure, network, throttling) propagates unhandled out of the route as a raw 500 instead of the app's structured `createApiError` JSON shape, inconsistent with the sibling upload-issue route. [`web/src/app/api/reviews/[id]/photos/confirm/route.ts:74`, `:98`]
- [x] [Review][Patch] Upload-issue route's `catch {}` around `getActiveReviewPhotoStorageStatus()`/`createUploadUrl()` discards the caught error entirely with no logging — turns any real R2/Supabase misconfiguration into a silent, undiagnosable 503 in production. [`web/src/app/api/reviews/[id]/photos/route.ts:72-74`]
- [x] [Review][Patch] Provider string comparisons in `resolveReviewPhotoStorageProviderStatus()` are case-sensitive with no enum validation — a typo like `R2` or `Supabase` in `REVIEW_PHOTO_STORAGE_PROVIDER` silently fails to match, defeating the intended override and the fallback-visibility warning. [`web/src/lib/reviews/storage/env.ts:58`, `:70`]
- [x] [Review][Patch] `R2_BUCKET_NAME` is listed in `R2_REQUIRED_ENV_VARS` even though `getR2BucketName()` already has a working default (`powri-review-photos`) — omitting it forces an unnecessary fallback to Supabase despite a valid default bucket existing. [`web/src/lib/reviews/storage/env.ts:8-14`, `:24-26`]
- [x] [Review][Patch] `getR2PublicBaseUrl()` only checks the value is non-empty and doesn't contain the S3 endpoint substring — never validates it's an actual well-formed URL. A malformed `R2_PUBLIC_BASE_URL` silently produces broken photo URLs shown to users. [`web/src/lib/reviews/storage/env.ts:28-39`]
- [x] [Review][Patch] R2 presigned PUT (`createUploadUrl`) has no enforced max content-length, and confirm never checks the downloaded byte size before running it through `sharp` — a client can PUT an arbitrarily large object directly to R2, bypassing the 20MB input-cap intent that Supabase enforces via bucket `file_size_limit`. Mitigate in code by rejecting with 400 when downloaded bytes exceed `MAX_REVIEW_PHOTO_INPUT_BYTES`, before resizing. [`web/src/lib/reviews/storage/r2Storage.ts:99-113`, `web/src/app/api/reviews/[id]/photos/confirm/route.ts:74`]
- [x] [Review][Patch] `extensionForReviewPhotoMime` in `validation.ts` is dead code — no longer called now that `buildReviewPhotoStoragePath` always appends `.webp` per AC 5; left over from the pre-WebP per-MIME-extension scheme and could mislead a future reader. Removed. [`web/src/lib/reviews/validation.ts:67-75`]
- [x] [Review][Patch] `storage/index.ts` (the provider-selection factory that AC 7/AC 10 depend on) had zero direct unit coverage — the only consumer test mocked `@/lib/reviews/storage` wholesale, so the R2/Supabase branching and the R2 singleton memoization were never exercised. Added `storage/index.test.ts`. [`web/src/lib/reviews/storage/index.ts`]
- [x] [Review][Defer] `deleteReviewPhotoRows` silently returns `{ storageRefs: [] }` and skips deletion when the initial `select` query errors — deferred, pre-existing silent-failure mode carried over unchanged from Story 12.2, not introduced by this story. [`web/src/lib/reviews/mutations.ts` — `deleteReviewPhotoRows`]
- [x] [Review][Defer] `isNotFoundError` treats any HTTP 404 status as "not found" (via the `$metadata.httpStatusCode === 404` fallback), which could also mask a genuinely wrong bucket/endpoint (e.g. `NoSuchBucket` also returns 404) as a benign "not yet migrated" state. [`web/src/lib/reviews/storage/r2Storage.ts:25-33`]
- [x] [Review][Defer] `SupabaseReviewPhotoStorage.createUploadUrl` ignores its `contentType` parameter while `R2ReviewPhotoStorage` pins `ContentType` on the presigned PUT — the shared `ReviewPhotoStorage` interface's content-type enforcement at upload time silently differs by provider. Low risk since the final compressed content-type is always set correctly at `overwrite()` regardless. [`web/src/lib/reviews/storage/supabaseStorage.ts` — `createUploadUrl`]
- [x] [Review][Defer] Full enforcement of a max-upload-size on the R2 bucket (companion to the patch above) needs an R2 bucket-level policy or a presigned-POST redesign with size-range conditions — infra/PO decision, not a pure code change. [`web/src/lib/reviews/storage/r2Storage.ts:99-113`]

---

## Dev Notes

### Brownfield (post Story 12.2)

| Area | Status |
|------|--------|
| Signed URL upload + confirm + resize | ✅ 12.2 — [`12-2-submit-edit-review-photos.md`](12-2-submit-edit-review-photos.md) |
| `review_photos.storage_path` | ✅ `{user_id}/{review_id}/{uuid}.{ext}` → **change ext to always `.webp`** for new uploads |
| Supabase bucket RLS | ✅ `002_storage_buckets.sql` — remains for legacy reads; no new writes after cutover |
| Storage abstraction | ❌ New |
| R2 bucket + credentials | ❌ PO to provision Cloudflare account before deploy |

### Files to UPDATE (read before editing)

| File | Current behavior | This story changes | Must preserve |
|------|------------------|-------------------|---------------|
| `web/src/lib/reviews/photoResize.ts` | Resize only if > 10 MB; keep original format; max edge 2048 | Always WebP; max edge 1600; target ≤ 2 MB | Return `null` on failure; pass-through semantics for already-small only if re-encode still ≤ 2 MB |
| `web/src/lib/reviews/validation.ts` | `MAX_REVIEW_PHOTO_BYTES = 10MB` | Set to 2 MB; path builder always `.webp` | Input cap 20 MB; max 5 photos |
| `web/src/app/api/reviews/[id]/photos/route.ts` | Supabase `createSignedUploadUrl` | Delegate to storage adapter; path ends `.webp` | Auth, rate limit, ownership, 403/401 shapes |
| `web/src/app/api/reviews/[id]/photos/confirm/route.ts` | Download/resize/overwrite via Supabase | Via adapter; set `storage_provider` on insert | Magic-byte check, path prefix validation, revalidate aggregate |
| `web/src/lib/reviews/mutations.ts` | All storage ops on `review-photos` bucket | Route through adapter | `revalidateReviewAggregate`, delete best-effort |
| `web/src/lib/reviews/photoUrl.ts` | Supabase public URL only | Dual-read by provider (+ null → supabase) | Path normalization |
| `web/src/lib/reviews/queries.ts` | Maps `storage_path` → URL | Pass `storage_provider` into URL builder | `REVIEW_LIST_SELECT` — add column |
| `web/next.config.ts` | Supabase hostname in `remotePatterns` | Add R2 public host when configured | Existing patterns |
| `web/src/lib/reviews/reviewApiClient.ts` | PUT to signed URL, confirm | **No change** if response shape unchanged | Background upload flow |

### Architecture compliance

- Phase 2 architecture §3.4 lists Supabase Storage for UGC — PO amendment 2026-08-11 moves **review photo blobs** to R2; Postgres metadata stays Supabase
- Confirm-route ownership check (`expectedPrefix = ${user.id}/${reviewId}/`) remains the security gate for R2 (no Storage RLS on R2)
- Service role: migration script uses `SUPABASE_SERVICE_ROLE_KEY` for downloads + R2 credentials for uploads — run server-side only, never expose R2 secret to client

### WebP path convention

```
Issue (POST /photos):  storagePath = {userId}/{reviewId}/{photoId}.webp
Client PUT:            original JPEG/PNG/WebP bytes to presigned URL
Confirm:               download → magic-byte validate → resize to WebP → overwrite same key
DB insert:             storage_path as issued; storage_provider = active provider
```

Do **not** keep original extension in path — PO confirmed `.webp` only.

### Proposed env vars

```bash
# Review photo storage (Story 12.2.1) — server-only except public base URL
REVIEW_PHOTO_STORAGE_PROVIDER=r2   # supabase | r2
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=powri-review-photos
R2_PUBLIC_BASE_URL=https://pub-xxxx.r2.dev
```

Factory fallback: if `REVIEW_PHOTO_STORAGE_PROVIDER=r2` but R2 vars missing, log warning and fall back to `supabase` (local dev safety).

R2 S3 endpoint: `https://{R2_ACCOUNT_ID}.r2.cloudflarestorage.com`

Install deps (latest, no invented pins): `@aws-sdk/client-s3`, `@aws-sdk/s3-request-presigner`

### Storage adapter interface (suggested)

```typescript
export type ReviewPhotoStorageProvider = 'supabase' | 'r2';

export interface ReviewPhotoStorage {
  provider: ReviewPhotoStorageProvider;
  createUploadUrl(storagePath: string): Promise<{ uploadUrl: string; token?: string }>;
  download(storagePath: string): Promise<Uint8Array | null>;
  overwrite(storagePath: string, bytes: Uint8Array, contentType: string): Promise<boolean>;
  delete(storagePaths: string[]): Promise<void>;
  publicUrl(storagePath: string): string;
}
```

### Migration script (`scripts/migrate-review-photos-to-r2.ts`)

1. Load env from `web/.env.local` (same pattern as other repo scripts)
2. SELECT `id, storage_path, storage_provider` FROM `review_photos` WHERE `storage_provider = 'supabase'`
3. For each row: download from Supabase → upload to R2 (same `storage_path`, preserve content-type from object metadata) → UPDATE `storage_provider = 'r2'`
4. Skip if HEAD/object exists in R2
5. Print summary: `{ migrated, skipped, failed }`
6. Exit non-zero if any failures

Add `"migrate:review-photos-r2": "tsx scripts/migrate-review-photos-to-r2.ts"` to root or `web/package.json` as appropriate.

**PO runs on staging first**, then production, as part of PR deploy checklist.

### Dual-read URL builder

```typescript
// photoUrl.ts — pseudocode
export function buildReviewPhotoPublicUrl(
  storagePath: string,
  provider: ReviewPhotoStorageProvider | null = 'supabase',
): string {
  if (provider === 'r2') {
    return `${R2_PUBLIC_BASE_URL}/${normalizedPath}`;
  }
  // existing Supabase URL
}
```

Update `mapReviewRow` to pass `photo.storage_provider` from SELECT.

### Story 12.2 carry-forward

- Background upload module-level queue survives navigation — do not move queue to component state
- `sharp` already installed — reuse for WebP encoding
- Bucket `file_size_limit` 20 MB in Supabase (`007_review_photos_input_cap.sql`) still applies when provider = supabase; R2 bucket should allow ≥ 20 MB PUT for pre-compress originals
- Account export (`lib/auth/accountExport.ts`) nests `storage_path` — paths remain valid strings; no export shape change required

### Out of scope

- Passport photos bucket migration (Epic 13)
- Avatar storage migration
- Custom domain + Cloudflare CDN cache rules
- Durable upload queue / service worker (12.2 accepted limitation unchanged)
- WebP for in-form preview thumbnails (display only)
- Deleting Supabase source objects post-migration (manual PO ops)
- Story **12.3** flat comments (deferred)

---

## Dev Agent Record

### Agent Model Used

Composer (dev-story 12.2.1)

### Debug Log References

- PO infra research + story draft approved 2026-08-11
- Open questions resolved: 12.2.1 before 12.4; 12.3 deferred; migration in same PR; `.webp` paths
- Build fix: kept `photoUrl.ts` client-safe (no storage adapter import) to avoid pulling `next/headers` into client bundle

### Completion Notes List

- Phase 0: confirm always re-encodes to WebP (1600px / q80 / ≤2 MB); storage paths always `.webp`
- Phase 1–2: `ReviewPhotoStorage` adapter with Supabase + R2 implementations; factory reads `REVIEW_PHOTO_STORAGE_PROVIDER` with credential fallback
- Phase 3: idempotent migration script `scripts/migrate-review-photos-to-r2.ts`; npm script `migrate:review-photos-r2` in `web/package.json`
- Dual-read URLs via `buildReviewPhotoPublicUrl(path, storage_provider)`; `REVIEW_LIST_SELECT` includes `storage_provider`
- Deletes route by row `storage_provider` for mixed legacy/R2 rows
- Verified: `npm run lint && npm run build && npm run test:unit` (338 tests pass)

**Correction (2026-08-11):** `resolveActiveReviewPhotoStorageProvider()` only warned on fallback when `REVIEW_PHOTO_STORAGE_PROVIDER=r2` was explicitly set. If that flag was left unset on Vercel and even one `R2_*` var was missing/mistyped, the provider silently fell back to `supabase` with zero log output — new blobs would land in the Supabase bucket with no visible error. Fixed to warn (naming the specific missing var) whenever any R2 config was attempted, not just when the flag was explicit.

### File List

- `web/src/lib/reviews/storage/env.ts`
- `web/src/lib/reviews/storage/env.test.ts`
- `web/src/lib/security/csp.ts`
- `web/src/lib/security/csp.test.ts`
- `supabase/migrations/008_review_photos_storage_provider.sql`
- `scripts/migrate-review-photos-to-r2.ts`
- `web/package.json`
- `web/package-lock.json`
- `web/.env.example`
- `web/next.config.ts`
- `docs/qa/phase2/deploy-environment.md`
- `web/src/lib/reviews/photoResize.ts`
- `web/src/lib/reviews/photoResize.test.ts`
- `web/src/lib/reviews/validation.ts`
- `web/src/lib/reviews/validation.test.ts`
- `web/src/lib/reviews/photoUrl.ts`
- `web/src/lib/reviews/photoUrl.test.ts`
- `web/src/lib/reviews/mutations.ts`
- `web/src/lib/reviews/queries.ts`
- `web/src/lib/reviews/reviewApiClient.ts`
- `web/src/lib/reviews/storage/types.ts`
- `web/src/lib/reviews/storage/env.ts`
- `web/src/lib/reviews/storage/env.test.ts`
- `web/src/lib/reviews/storage/supabaseStorage.ts`
- `web/src/lib/reviews/storage/supabaseStorage.test.ts`
- `web/src/lib/reviews/storage/r2Storage.ts`
- `web/src/lib/reviews/storage/r2Storage.test.ts`
- `web/src/lib/reviews/storage/index.ts`
- `web/src/lib/reviews/storage/index.test.ts`
- `web/src/app/api/reviews/[id]/photos/route.ts`
- `web/src/app/api/reviews/[id]/photos/route.test.ts`
- `web/src/app/api/reviews/[id]/photos/confirm/route.ts`
- `web/src/app/api/reviews/[id]/photos/confirm/route.test.ts`
- `web/src/app/api/reviews/[id]/route.ts`

### Change Log

- 2026-08-11: Story 12.2.1 created — R2 migration + WebP compression (ready-for-dev).
- 2026-08-11: Implementation complete — WebP compression, R2 storage adapter, migration script, dual-read URLs (review).
- 2026-08-13: Round-2 code review (post manual-QA) — patched unhandled R2 download errors in confirm route (503 instead of raw 500), logged the previously-swallowed error in the upload-issue route's catch block, made `REVIEW_PHOTO_STORAGE_PROVIDER` comparisons case-insensitive, dropped the unnecessary `R2_BUCKET_NAME` requirement (has a working default), added well-formed-URL validation to `getR2PublicBaseUrl()`, and added an input-size cap check on downloaded bytes before compression to mitigate unbounded R2 PUT size. 4 additional issues deferred (pre-existing or infra-level) — see `deferred-work.md`. `npm run lint && npm run build && npm run test:unit` pass (368 tests). A follow-up Acceptance Auditor pass then surfaced two more low-severity gaps, also fixed: removed dead `extensionForReviewPhotoMime` from `validation.ts`, and added direct unit coverage for the `storage/index.ts` provider-selection factory (`storage/index.test.ts`). Final verification: 373 tests pass.
