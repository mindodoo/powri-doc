---
baseline_commit: f915d28c8ab16f6688a5ec96c1d1781446702eb4
---

# Story 12.2: Submit & Edit Review with Photos

Status: done

## Story

As a **signed-in visitor**,  
I want to write one review per resort with optional photos,  
So that I can share my experience publicly.

**Source shard(s):** [`epics/phase2/shards/epic-12-community-reviews.md`](../../planning-artifacts/epics/phase2/shards/epic-12-community-reviews.md) · [`prds/phase2/shards/06-features-reviews-comments.md`](../../planning-artifacts/prds/phase2/shards/06-features-reviews-comments.md) · UX [`ux-designs/ux-phase2/experience.md`](../../planning-artifacts/ux-designs/ux-phase2/experience.md) · [`design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §ExperienceSection · addendum [`§B–§M`](../../planning-artifacts/prds/phase2/addendum.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Story **12.1** (Experience section, `GET /api/reviews`, aggregate SSR, `WriteReviewButton` shell). Epic **9** auth + Supabase schema (`reviews`, `review_photos`, storage bucket `review-photos`).

**Blocks:** Story **12.3** (comments below reviews), **12.4** (Report kebab item on non-owner cards). Epic **13.2** badge evaluation post-review (explicitly **out of scope** this story).

**Epic note:** Story **15.3** gates public UGC launch legally — dev can ship to staging before that.

---

## PO decisions (confirmed 2026-08-07)

| Topic | Decision |
|-------|----------|
| **PRD reference** | [`06-features-reviews-comments.md`](../../planning-artifacts/prds/phase2/shards/06-features-reviews-comments.md) — **not** AI chat (Phase 3 AI theme summaries remain out of scope) |
| **Form UI — mobile** | **New page** at `/resorts/[slug]/review` (full-screen form, not bottom sheet) |
| **Form UI — desktop (≥768px)** | **Modal** overlay on resort detail (reuse form fields; do not navigate away) |
| **Post sign-in resume** | **Auto-open** form after auth return (addendum §A return-to-intent): mobile → land on `/resorts/[slug]/review`; desktop → return to detail with `?review=write` (or equivalent) and open modal |
| **Create entry** | **Write a review** CTA (guest → sign-in; signed-in without existing review → form) |
| **Edit / delete entry** | **Kebab menu** on owner's `ReviewCard` only — **Edit** opens form (page mobile / modal desktop); **Delete** soft-deletes review |
| **One review per resort** | DB unique `(user_id, resort_slug)` — **hide Write CTA** when signed-in user already has a published review on that resort |
| **Photo upload** | **Signed URL flow** per architecture — `POST /api/reviews/[id]/photos` issues upload URL; client uploads direct to Supabase Storage |
| **Photo edit** | **Remove individual photos + add new** — still **max 5** total; reorder optional (preserve `sort_order` on save) |
| **Delete review** | **Soft delete** — set `reviews.status = 'deleted'`; remove from public list; clean up storage objects for removed photos on edit/delete |
| **Badge evaluation** | **Deferred to Epic 13.2** — do not call `/api/badges/evaluate` or show unlock toast in 12.2 |
| **Analytics — create** | `review_submitted` with `resort_slug`, `rating`, `photo_count` (first publish only) |
| **Analytics — update** | **New** `review_edited` with `resort_slug`, `rating`, `photo_count` (tracking-plan + `phase2Events.ts` update required) |
| **Analytics — delete** | **New** `review_deleted` with `resort_slug` (tracking-plan + registry update; mirrors separate-event pattern for edits) |
| **WebP resize pipeline** | **Out of scope** — addendum §B step 3 optional; validate MIME via magic bytes, enforce 10 MB, store with detected extension under `{user_id}/{review_id}/{uuid}.{ext}` |
| **Profanity on review body** | **Not required** Phase 2 — report queue handles abuse (addendum §O applies to display name/bio only today) |
| **Zero-review copy** | **`countZero` only** in aggregate — no `emptyState` line in `ReviewList` (PO 2026-08-10; avoids duplicate "No reviews yet" messaging) |

---

## Acceptance Criteria

### A. Write review entry & auth gate

1. **Given** a **guest** on resort detail  
   **When** they tap **Write a review**  
   **Then** `SignInSheet` opens with `trigger: 'review'` and `returnTo` set per breakpoint:
   - **Mobile:** `/resorts/{slug}/review`
   - **Desktop:** `/resorts/{slug}?review=write`

2. **Given** a **signed-in user with no review** for this resort  
   **When** they tap **Write a review**  
   **Then** the review form opens:
   - **Mobile:** navigate to `/resorts/{slug}/review`
   - **Desktop:** open **ReviewFormModal** on current detail page

3. **Given** a **signed-in user who already published a review** for this resort  
   **When** they view the Experience section  
   **Then** **Write a review** CTA is **hidden** (edit/delete only via owner kebab on their card)

4. **Given** auth completes with the return URLs above  
   **When** the user lands back on the app  
   **Then** the review form **auto-opens** (mobile: review page; desktop: modal from query flag)

### B. Review form (create & edit)

5. **Given** the review form (page or modal)  
   **When** rendered  
   **Then** fields include:
   - **Star rating** (1–5, required, tap to select)
   - **Body text** (required, max **2,000** chars, show counter)
   - **Photo attach** (optional, up to **5**, max **10 MB** each, JPEG/PNG/WebP)

6. **And** submit button disabled until rating + non-empty trimmed body  
7. **And** while submitting or uploading photos, submit is disabled and shows progress state (UX: "Review submitting")  
8. **And** client validates photo count/size/MIME **before** upload; inline errors for violations  
9. **And** edit mode pre-fills rating, body, and existing photo thumbnails with remove control per photo

### C. Create review API

10. **Given** an authenticated user with **no** existing review for `resortSlug`  
    **When** `POST /api/reviews` with `{ resortSlug, rating, body }`  
    **Then** response **201** with created review `{ id, ... }`  
    **And** row inserted with `status = 'published'`, `user_id = auth.uid()`  
    **And** body sanitized via `lib/ugc/sanitize.ts` (strip HTML, enforce max length)  
    **And** `rating` validated 1–5 integer

11. **Given** user already has a review for that resort  
    **When** `POST /api/reviews` for same slug  
    **Then** **409 CONFLICT** with clear message (use existing review / edit instead)

12. **And** route requires auth — **401** if no session  
13. **And** `ugcWrite` rate limit **20/min/user** (extend `RATE_LIMIT_POLICIES` per architecture §5)

### D. Edit & soft-delete API

14. **Given** the review owner  
    **When** `PATCH /api/reviews/[id]` with `{ rating?, body? }`  
    **Then** **200** with updated review  
    **And** only owner can patch (RLS + handler check); **403** otherwise  
    **And** `updated_at` refreshed; sanitized body same rules as create

15. **Given** the review owner  
    **When** `PATCH /api/reviews/[id]` with `{ status: 'deleted' }` (or dedicated delete semantics documented in handler)  
    **Then** review `status` becomes **`deleted`**  
    **And** review no longer returned by public `GET /api/reviews`  
    **And** associated `review_photos` rows removed and storage objects deleted (best-effort cleanup)

16. **Given** a non-owner  
    **When** PATCH including delete  
    **Then** **403**

### E. Photo upload (signed URL)

17. **Given** review owner and review `id`  
    **When** `POST /api/reviews/[id]/photos` with `{ contentType, contentLength, sortOrder }`  
    **Then** **200** with `{ uploadUrl, storagePath, photoId }` for Supabase signed upload  
    **And** total photos for review ≤ **5** (count existing + pending)  
    **And** `contentLength` ≤ 10 MB; `contentType` in `image/jpeg|png|webp`

18. **Given** client receives signed URL  
    **When** upload completes  
    **Then** client calls confirm step (same POST response includes DB row **or** follow-up PATCH) so `review_photos` row exists with `storage_path`, `sort_order`  
    **And** storage path matches `{user_id}/{review_id}/{uuid}.{ext}`

19. **Given** owner removes photo in edit UI  
    **When** save  
    **Then** `DELETE` handler or PATCH payload removes `review_photos` row and deletes storage object  
    **And** remaining photos keep valid `sort_order`

20. **And** upload route enforces `upload` rate limit **10/min/user** (architecture §5)

### F. Owner kebab menu on ReviewCard

21. **Given** signed-in user viewing **their own** review in the list  
    **When** the card renders  
    **Then** kebab/menu shows **Edit** and **Delete** (Report **not** shown — Story 12.4)

22. **Given** signed-in user viewing **another user's** review  
    **Then** no owner kebab in 12.2 (Report arrives in 12.4)

23. **Given** owner taps **Edit**  
    **Then** same form as create opens pre-filled (mobile page / desktop modal)

24. **Given** owner taps **Delete**  
    **Then** confirm dialog → soft delete → card removed from list + aggregate refresh on next navigation/revalidate

### G. Post-submit UX & cache

25. **Given** successful **create**  
    **When** form closes  
    **Then** new review appears at top of client list (optimistic refresh or refetch)  
    **And** SSR aggregate revalidates within 60s TTL (`revalidatePath` or tag for resort detail experience)

26. **Given** successful **edit**  
    **Then** card updates in place with edited label when `updated_at > created_at` (12.1 date helper already supports "edited")

### H. Analytics

27. **Given** first successful create  
    **Then** fire `review_submitted` with `resort_slug`, `rating`, `photo_count`

28. **Given** successful edit (rating/body/photos changed)  
    **Then** fire `review_edited` with same properties

29. **Given** successful soft delete  
    **Then** fire `review_deleted` with `resort_slug`

30. **And** update [`docs/analytics/tracking-plan.md`](../../../docs/analytics/tracking-plan.md) §3 + [`web/src/lib/analytics/phase2Events.ts`](../../../web/src/lib/analytics/phase2Events.ts) for `review_edited` and `review_deleted`  
31. **And** `npm run test:analytics` passes

### I. Sanitization & security (NFR-10, NFR-12)

32. **And** `web/src/lib/ugc/sanitize.ts` created — strip HTML tags, trim, enforce max lengths (review body 2000)  
33. **And** server re-validates photo MIME with magic-byte check (reuse avatar byte detection pattern from `validateAvatarBytes.ts`)  
34. **And** no `SUPABASE_SERVICE_ROLE_KEY` on public read paths; signed uploads use authenticated user + RLS storage policies

### J. i18n & accessibility

35. **And** strings under `messages/en.json` → `resortDetail.reviewForm.*`, `resortDetail.experience.kebab.*`  
36. **And** form fields have labels; star input has accessible name; modal traps focus and closes on Esc (desktop)  
37. **And** photo remove buttons have accessible names; upload errors announced

### K. UX polish (post-QA fixes — approved 2026-08-07)

38. **Given** the Experience section renders  
    **When** the resort has **zero** published reviews  
    **Then** only **one** zero-count message appears — `countZero` ("No reviews yet") in the SSR aggregate (`AggregateRating`)  
    **And** only **one** "Write a review" CTA appears in `ReviewList` (no second zero-count line in the list)  
    **And** the separate CTA previously rendered directly under the aggregate is **not** shown in this case  
    **PO amendment 2026-08-10:** `emptyState` ("No reviews yet — be the first") removed — redundant with `countZero` when both were shown during QA

39. **Given** the Experience section renders  
    **When** the resort has **one or more** published reviews **and** the signed-in user has no review of their own  
    **Then** the single "Write a review" CTA appears above the review cards (existing top-of-section placement, next to the aggregate) — this is the only CTA shown in this case

40. **Given** the review form (page or modal)  
    **When** rendered  
    **Then** the photo attach control is a visible **"Add photos"** button (not a bare native file input) — the native input is visually hidden but still triggered by the button and remains keyboard/screen-reader accessible  
    **And** layout order top-to-bottom is: **Add photos** button → hint text ("Add up to 5 photos, 10 MB each — JPG, PNG, or WebP") → selected/existing photo thumbnails

41. **Given** desktop (≥768px) `ReviewFormModal`  
    **When** rendered  
    **Then** modal width is widened to a landscape-appropriate size (~640–700px single column) instead of the current narrow portrait sizing (was `max-w-[560px]`, observed rendering ~332×618)

42. **And** the photo count cap (max 5) enforcement at selection time keeps existing behavior — trims a selected batch to remaining slots and shows an inline message for any dropped files; AC 40's visual redesign is what makes this affordance noticeable (no new enforcement mechanism required)

43. **Given** a selected photo exceeds 10 MB  
    **When** validated  
    **Then** the photo is **not** rejected outright — the server resizes/compresses it (max long edge, reduced quality) to fit under the effective storage limit before saving the `review_photos` row  
    **And** the client-side pre-check ceiling is relaxed to allow larger originals up to a new input cap (see Dev Notes for proposed default)  
    **And** if resizing still cannot bring the file under the limit, the photo is rejected with a clear message

44. **Given** a published review has photos  
    **When** `ReviewCard` renders them  
    **Then** photos display in a **2×2 square-cropped grid** (`object-cover`) instead of the current horizontal scroll strip  
    **And** if there are more than 4 photos, the 4th tile shows a `+N` overlay (N = remaining count) anchored bottom-right  
    **And** each tile is clickable

45. **Given** a user clicks/taps a review photo tile  
    **Then** a lightbox opens:
    - **Mobile:** full-screen viewer
    - **Desktop:** centered modal with an **"X"** close control
    **And** the clicked photo is shown at its original aspect ratio (letterboxed with black background filling any remaining viewer area)  
    **And** the user can navigate to the review's other photos via swipe (mobile) or arrow controls (desktop)

46. **And** the photo grid/lightbox redesign (AC 44–45) applies only to the **published review display** (`ReviewCard`) — the in-form photo preview while composing/editing (`ReviewForm`) is unchanged (simple thumbnail list)

47. **Given** a review submission includes new photos  
    **When** the review text/rating save succeeds  
    **Then** the form closes and the review appears in the list **immediately**, without waiting for photo uploads to finish  
    **And** photo uploads continue in the background  
    **And** the user sees a message that photos may take a few minutes to appear  
    **And** (accepted limitation, v1) if the user closes the tab or fully navigates away before background uploads finish, those in-progress photo uploads may be lost silently — best-effort only, no durable queue/service worker

48. **And** `review_submitted` / `review_edited` analytics fire once with the final `photo_count` after background uploads settle (see Dev Notes — exact timing to confirm at implementation)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `lib/ugc/sanitize.ts`; review validation helpers; photo limit/count logic; signed-path builder
- [x] **API route tests:** POST create (201, 409 duplicate, 401, validation); photo confirm (resize/MIME). PATCH handler covered by PO manual QA + text-only E2E (no dedicated `[id]/route.test.ts`)
- [x] **Analytics:** `docs/analytics/tracking-plan.md` + `phase2Events.ts` + `npm run test:analytics`
- [x] **Content / resorts:** `npm run test:launch` — build/SSG for 20 resorts succeeds; script exits 1 locally on pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env gate only (Story 12.1 carry-forward)
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **E2E:** `web/e2e/review-text-crud.spec.ts` — mobile page + desktop modal create/edit/delete (text-only, no photos); skipped in CI when Supabase keys are placeholders
- [x] **Manual QA (PO):** guest Write → sign-in → auto form; create text-only; create with 3 photos; edit text + remove 1 photo + add 1; delete with confirm; 6th photo rejected client+server; duplicate POST 409; non-owner PATCH 403; aggregate count updates after create/delete
- [x] **Manual QA (PO) — UX polish:** zero-review resort shows exactly one Write CTA; resort with existing reviews (no own review) shows exactly one Write CTA above cards; "Add photos" button visible and obvious; desktop modal renders landscape-proportioned; select 8 files at once → only 5 added + inline message; upload a >10MB photo → accepted and resized, not rejected; review with 5 photos shows 2×2 grid with "+1" overlay; tapping a photo opens lightbox (full-screen mobile / modal+X desktop) with working carousel nav; submit review with photos → form closes immediately, "may take a few minutes" message shown, photos appear later without reload

---

## Tasks / Subtasks

- [x] `web/src/lib/ugc/sanitize.ts` + tests (AC: 10, 14, 32)
- [x] Extend `web/src/lib/rate-limit/limiter.ts` — add `ugcWrite` (20/min) and `upload` (10/min) policies (AC: 13, 20)
- [x] `web/src/lib/reviews/validation.ts` — rating/body/photo request schemas (Zod) (AC: 10–11, 14–17)
- [x] `web/src/app/api/reviews/route.ts` — add **POST** handler (AC: 10–13)
- [x] `web/src/app/api/reviews/[id]/route.ts` — **PATCH** edit + soft delete (AC: 14–16)
- [x] `web/src/app/api/reviews/[id]/photos/route.ts` — signed URL + confirm + delete photo (AC: 17–20)
- [x] `web/src/components/experience/ReviewForm.tsx` — shared form fields (AC: 5–9)
- [x] `web/src/components/experience/ReviewFormModal.tsx` — desktop modal wrapper (AC: 2, 4)
- [x] `web/src/app/[locale]/resorts/[slug]/review/page.tsx` — mobile full-page form (AC: 1–2, 4)
- [x] Update `WriteReviewButton.tsx` — breakpoint-aware navigation vs modal trigger; hide when user has review (AC: 1–3, 4)
- [x] Update `ReviewCard.tsx` — owner kebab Edit/Delete (AC: 21–24)
- [x] Update `ReviewList.tsx` — pass current user id; refetch after mutations; desktop modal host + query param listener for `?review=write` (AC: 25–26)
- [x] Post-auth auto-open wiring in review page + detail client island (AC: 4)
- [x] Analytics: `review_submitted`, `review_edited`, `review_deleted` + tracking plan (AC: 27–31)
- [x] i18n keys (AC: 35–37)
- [x] Verification commands (Testing section)

### UX Polish Tasks (post-QA, approved 2026-08-07 — implemented)

- [x] Consolidate "Write a review" CTA into `ReviewList` as single source of truth (`total` + `showWriteButton`); remove standalone top-level render in `ExperienceSectionClient` (AC: 38–39)
- [x] Redesign photo attach control: visually hidden native input + visible "Add photos" button; reorder to button → hint → thumbnails (AC: 40)
- [x] Widen `ReviewFormModal` desktop width to ~640–700px single column (AC: 41)
- [x] Verify AC 42 (selection cap) still holds after AC 40 redesign — no new logic expected, confirm via manual QA
- [x] Add server-side image resize on photo confirm (new dependency, e.g. `sharp`); relax client max-size pre-check; resize oversized images to fit under storage limit (AC: 43)
- [x] Redesign `ReviewCard` photo display: 2×2 grid, square crop, `+N` overlay on 4th tile, clickable tiles (AC: 44)
- [x] Build lightbox/full-screen photo viewer with carousel navigation (mobile full-screen, desktop modal with close) (AC: 45–46)
- [x] Implement background/optimistic photo upload: close form after review text saves, continue photo uploads in background, show "may take a few minutes" messaging; fire analytics after uploads settle (AC: 47–48)
- [x] Update i18n strings for new copy (Add photos button, background upload messaging, lightbox controls)
- [x] Extend unit tests + manual QA checklist for new behaviors

---

## Dev Notes

### Brownfield (post Story 12.1)

| Area | Status |
|------|--------|
| `GET /api/reviews` | ✅ 12.1 |
| `ExperienceSection` + list + aggregate | ✅ 12.1 |
| `WriteReviewButton` | ✅ Disabled stub for signed-in — **replace** with real behavior |
| `ReviewCard` | ✅ Read-only — **add** owner kebab |
| `lib/ugc/sanitize.ts` | ❌ Not implemented (only `profanity.ts` exists) |
| POST/PATCH review routes | ❌ |
| Photo signed URL route | ❌ |
| `/resorts/[slug]/review` page | ❌ |
| `ugcWrite` / `upload` rate limits | ❌ Not in `RATE_LIMIT_POLICIES` yet |

**Reuse:** `jsonOk` / `createApiError`, `requireUser`, `createClient()`, avatar MIME detection (`validateAvatarBytes.ts`), `photoUrl.ts` for public URLs, `SignInSheet` / `openSignIn`, design tokens from Phase 2 `design.md`.

### Architecture compliance

- [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §5 API table · §6.2 `ReviewForm` in `components/experience/` · §6.6 aggregate `revalidate: 60`
- **Create flow order:** (1) POST review text+rates → (2) for each new photo POST signed URL → client upload → confirm DB row → (3) track analytics → (4) refresh list + revalidate aggregate
- **Edit flow:** PATCH review fields; photo adds via same signed URL route; photo removes via DELETE on photo id or PATCH with `removedPhotoIds`
- **Do not** call `lib/badges/evaluate.ts` (13.2)

### Signed URL implementation hint

Supabase JS v2: `createClient().storage.from('review-photos').createSignedUploadUrl(path)` (authenticated). Handler validates review ownership before issuing URL. Path: `` `${user.id}/${reviewId}/${crypto.randomUUID()}.${ext}` ``.

Confirm step: insert `review_photos` after successful upload (client reports back `storagePath` + `sortOrder`; server verifies object exists in bucket).

### Desktop modal + query param pattern

Detail page client wrapper (e.g. extend `ReviewList` or small `ExperienceWriteHost`):

```typescript
// On mount: if searchParams.get('review') === 'write' && isAuthenticated && !hasOwnReview → open modal
// On modal close: router.replace(pathname) to strip query param
```

Mobile `WriteReviewButton` uses `<Link href={`/resorts/${slug}/review`}>` when authenticated.

Guest sign-in `returnTo` must use paths above so post-auth redirect opens form automatically.

### API shapes (suggested)

```typescript
// POST /api/reviews
{ resortSlug: string; rating: number; body: string }

// PATCH /api/reviews/[id]
{ rating?: number; body?: string; status?: 'deleted' }

// POST /api/reviews/[id]/photos
{ contentType: 'image/jpeg' | 'image/png' | 'image/webp'; contentLength: number; sortOrder: number }
// → { photoId, storagePath, uploadUrl, token? }

// DELETE /api/reviews/[id]/photos/[photoId]  (optional dedicated route for remove-on-edit)
```

### Detecting "user already has review"

Option A (preferred): authenticated `GET /api/reviews/me?resortSlug=` returning `{ review: PublicReview | null }` — avoids scanning full list.

Option B: client filters public list by `author.username` vs session profile — works but wasteful; use only if avoiding new route.

### Story 12.1 carry-forward

- Half-star **aggregate** display stays; per-review stars remain integer 1–5 in form
- Migration `005_review_photos_public_read.sql` already applied for anon photo read
- `test:launch` env gate (`NEXT_PUBLIC_CONTACT_EMAIL`) pre-existing — not introduced by 12.2

### UX polish — technical notes (post-QA, approved 2026-08-07)

PO decisions from manual QA follow-up:

| Topic | Decision |
|-------|----------|
| **CTA duplication** | Keep top CTA only when review list is **non-empty**; when empty, aggregate `countZero` + list Write CTA only (no `emptyState` list copy) |
| **Modal sizing** | Wider **single-column** modal (~640–700px), not a two-column layout |
| **Selection cap enforcement** | Keep current trim-and-message behavior; no stricter enforcement needed once "Add photos" button makes the control visible |
| **Oversized photo handling** | **Server-side** resize (new dependency, e.g. `sharp`) rather than client-side canvas resize |
| **Gallery/lightbox scope** | `ReviewCard` published display only — `ReviewForm` in-progress preview thumbnails unchanged |
| **Lightbox navigation** | Carousel — swipe (mobile) / arrows (desktop) between a review's photos |
| **Upload latency fix** | Background/optimistic submission now (not deferred pending resize-only fix) |
| **Background upload data-loss risk** | Accepted for v1 — best-effort only, no durable queue/service worker; lost uploads if tab closes before completion |
| **Zero-review copy** | **Single line only** — `countZero` in aggregate; no `emptyState` in `ReviewList` (PO 2026-08-10 — duplicate messaging rejected in QA) |

Implementation notes / proposed defaults (confirm during implementation if anything below seems wrong):

- **CTA consolidation:** move the `total === 0` vs `> 0` branching into `ReviewList` (it already receives `showEmptyWriteButton`); `ExperienceSectionClient` stops rendering `WriteReviewButton` directly above `ReviewList`.
- **Modal width:** propose `md:max-w-[680px]` (up from `max-w-[560px]`) on `ReviewFormModal`.
- **Resize (AC 43) proposed defaults:**
  - Relax client pre-check ceiling from 10MB to a new input cap (proposed **20MB**) — still hard-reject above this.
  - Server resizes any image validated >10MB (after existing magic-byte check) to: max long edge **2048px**, output **JPEG quality ~82**; re-check resulting size ≤10MB before inserting the `review_photos` row.
  - New dependency: `sharp` — install via npm per `AGENTS.md` dependency rule (latest version, no invented version pins).
- **Gallery grid (AC 44):** define explicit layouts for 1–3 photos (not just the 4+ case) during implementation — e.g. single photo full-width square, 2 photos side-by-side, 3 photos in an asymmetric grid — to avoid an awkward half-empty 2×2 grid.
- **Background upload (AC 47–48):** review text/rating POST resolves and closes the form immediately; photo uploads (issue → PUT → confirm per photo) continue via a fire-and-forget async task in the same page session. Recommend firing `review_submitted`/`review_edited` once, after background uploads settle (success or partial failure), so `photo_count` reflects the true outcome rather than an optimistic count — flag if immediate submit-time count is preferred instead for funnel-timing reasons.
- **Out-of-scope carry-forward:** blanket WebP conversion of all uploads remains out of scope — resize only triggers for photos that would otherwise exceed 10MB.

### Out of scope (explicit)

- Flat comment thread (**12.3**)
- Report kebab + admin hide (**12.4**)
- Badge evaluation + unlock toast (**13.2**)
- AI theme summaries (Phase 3 — [`phase-3-user-community.md`](../../planning-artifacts/epics/phase1/shards/phase-3-user-community.md))
- WebP resize/convert pipeline (addendum §B optional step 3)
- Profanity filter on review body
- Community guidelines legal page (**15.3**)

---

## Dev Agent Record

### Agent Model Used

Composer (Cursor)

### Debug Log References

- PO clarifications 2026-08-07: mobile page + desktop modal; signed URL photos; kebab edit/delete; soft delete; photo remove+add; badge eval deferred; separate analytics events; post-sign-in auto-open; reviews PRD not AI chat.
- PO clarifications 2026-08-07 (post-QA UX polish, second round): top-CTA suppressed only when list empty; wider single-column modal; keep trim-and-message selection cap; server-side resize (new `sharp` dependency) for >10MB photos; 2×2 grid + carousel lightbox on `ReviewCard` only; background/optimistic photo upload accepted with best-effort data-loss risk for v1. Approved — added to story as AC 38–48 + new task list; **implementation not started**, pending explicit go-ahead.
- 2026-08-07 (UX polish implementation): discovered the `review-photos` Supabase Storage bucket has a server-enforced `file_size_limit` of 10MB (`002_storage_buckets.sql`). Raising the client pre-check to 20MB (AC 43) would have originals between 10–20MB rejected at the raw signed-URL `PUT` before ever reaching the confirm/resize step — added migration `007_review_photos_input_cap.sql` to raise the bucket limit to 20MB so those uploads land and the server-side resize logic can run.
- Background upload (AC 47–48) implemented as a module-level pub/sub queue (`backgroundPhotoUpload.ts`) rather than component state, since the mobile flow navigates away from the review page (unmounting the form) immediately after the text/rating save. Module state survives client-side route changes, so uploads + the once-settled analytics call complete even after navigating back to the resort detail page; `ReviewList` subscribes to show the "uploading" message and silently refetch when done. Accepted limitation unchanged: a full tab close/reload before uploads settle still drops in-flight uploads (best-effort, no durable queue).

### Completion Notes List

- PO clarifications 2026-08-07 applied: `GET /api/reviews/me`, reuse-row re-publish after soft delete, two-step photo confirm, PATCH `removedPhotoIds`, migration 006 RLS, magic-byte check at confirm.
- Implemented create/edit/delete review flows with signed URL photo upload, desktop modal + mobile page, owner kebab, analytics events.
- Verification: `npm run lint`, `npm run build`, `npm run test:unit`, `npm run test:analytics` pass. `test:launch` fails on pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env gate (Story 12.1 note).
- **UX polish (AC 38–48) implemented 2026-08-07:**
  - AC 38–39: `ReviewList` is now the single source of truth for the "Write a review" CTA (`showWriteButton` prop) — one CTA when list empty (aggregate `countZero` carries zero-count copy), one CTA above the cards when reviews exist; `ExperienceSectionClient` no longer renders its own copy.
  - AC 40: `ReviewForm` photo attach is now a visible "Add photos" button that triggers a visually-hidden (`sr-only`, still focusable) native file input; layout order is button → hint → thumbnails.
  - AC 41: `ReviewFormModal` desktop width widened to `md:max-w-[680px]` single column.
  - AC 42: confirmed the existing trim-and-message selection-cap logic in `ReviewForm.handleAddPhotos` needs no changes after the AC 40 redesign.
  - AC 43: added `sharp` and `web/src/lib/reviews/photoResize.ts` (server-side resize: max long edge 2048px, quality ~82, re-encoded in the original format); wired into the photo confirm route (`resizeReviewPhotoIfNeeded` → `overwriteReviewPhotoBytes` on the same storage path when resized). Client cap relaxed from 10MB to a new `MAX_REVIEW_PHOTO_INPUT_BYTES` (20MB) in `validation.ts`/`clientForm.ts`. Added migration `007_review_photos_input_cap.sql` to raise the Storage bucket's own `file_size_limit` to 20MB (see Debug Log).
  - AC 44: new `ReviewPhotoGrid` component renders explicit layouts for 1 photo (single square), 2 photos (side-by-side), 3 photos (asymmetric 2×2 with one tall tile), and 4+ photos (2×2 grid with a `+N` overlay on the 4th tile when more than 4 exist).
  - AC 45–46: new `ReviewPhotoLightbox` component — full-screen on mobile, centered modal with an "X" close on desktop (`md:` breakpoint), image shown with `object-contain` letterboxed on black, swipe (touch) and arrow-button (desktop) carousel navigation across the review's full photo list. Wired only into `ReviewCard`; `ReviewForm`'s in-progress thumbnail preview is unchanged.
  - AC 47–48: `ReviewForm.handleSubmit` now saves review text/rating first and calls `onSuccess` immediately; any new photos are handed to `enqueueBackgroundPhotoUpload` (new `backgroundPhotoUpload.ts` module-level queue) which uploads them one by one (best-effort, continues past individual failures) and fires `review_submitted`/`review_edited` exactly once after all uploads settle, using the true final `photo_count`. `ReviewList` subscribes to the queue to show an "uploading, may take a few minutes" message and silently refresh the list once uploads for that resort settle — this works across the mobile page→detail navigation because the queue lives at module scope, which survives client-side route changes.
- Added new tests: `clientForm.test.ts`, `photoResize.test.ts`, `backgroundPhotoUpload.test.ts`, `[id]/photos/confirm/route.test.ts`, plus new cases in `validation.test.ts` for the relaxed upload cap.
- Re-verified after polish changes: `npm run lint`, `npm run build`, `npm run test:unit` (330 tests, 81 files), `npm run test:analytics` all pass. `test:launch` still fails only on the same pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env gate (unrelated to this story).
- **Code review + E2E (2026-08-10):** `web/e2e/review-text-crud.spec.ts` added (text-only create/edit/delete, mobile + desktop); 2/2 pass locally with real Supabase keys. `supabaseSession.ts` cookie port default aligned to Playwright `3012`. PO confirmed `countZero`-only for zero reviews — removed unused `emptyState` i18n key.

### File List

- `supabase/migrations/006_review_photos_owner_write.sql`
- `supabase/migrations/007_review_photos_input_cap.sql` (UX polish — AC 43)
- `web/src/lib/ugc/sanitize.ts`
- `web/src/lib/ugc/sanitize.test.ts`
- `web/src/lib/rate-limit/limiter.ts`
- `web/src/lib/rate-limit/enforceUserRateLimit.ts`
- `web/src/lib/reviews/validation.ts` (UX polish — `MAX_REVIEW_PHOTO_INPUT_BYTES`, AC 43)
- `web/src/lib/reviews/validation.test.ts` (UX polish — relaxed cap cases)
- `web/src/lib/reviews/mutations.ts` (UX polish — `overwriteReviewPhotoBytes`, AC 43)
- `web/src/lib/reviews/photoBytes.ts`
- `web/src/lib/reviews/photoResize.ts` (new — UX polish, AC 43)
- `web/src/lib/reviews/photoResize.test.ts` (new — UX polish)
- `web/src/lib/reviews/clientForm.ts` (UX polish — relaxed client cap, AC 43)
- `web/src/lib/reviews/clientForm.test.ts` (new — UX polish)
- `web/src/lib/reviews/reviewApiClient.ts` (UX polish — export `uploadReviewPhoto`, remove blocking `submitReviewWithPhotos`, AC 47–48)
- `web/src/lib/reviews/backgroundPhotoUpload.ts` (new — UX polish, AC 47–48)
- `web/src/lib/reviews/backgroundPhotoUpload.test.ts` (new — UX polish)
- `web/src/app/api/reviews/route.ts`
- `web/src/app/api/reviews/route.post.test.ts`
- `web/src/app/api/reviews/me/route.ts`
- `web/src/app/api/reviews/[id]/route.ts`
- `web/src/app/api/reviews/[id]/photos/route.ts`
- `web/src/app/api/reviews/[id]/photos/confirm/route.ts` (UX polish — server-side resize wiring, AC 43)
- `web/src/app/api/reviews/[id]/photos/confirm/route.test.ts` (new — UX polish)
- `web/src/components/experience/StarRatingInput.tsx`
- `web/src/components/experience/ReviewForm.tsx` (UX polish — Add photos button, background upload, AC 40/47–48)
- `web/src/components/experience/ReviewFormModal.tsx` (UX polish — wider desktop modal, AC 41)
- `web/src/components/experience/DeleteReviewConfirmDialog.tsx`
- `web/src/components/experience/ExperienceSectionClient.tsx` (UX polish — CTA consolidation, AC 38–39)
- `web/src/components/experience/ExperienceSection.tsx`
- `web/src/components/experience/ExperienceAggregate.tsx`
- `web/src/components/experience/WriteReviewButton.tsx`
- `web/src/components/experience/ReviewCard.tsx` (UX polish — photo grid + lightbox wiring, AC 44–46)
- `web/src/components/experience/ReviewList.tsx` (UX polish — CTA consolidation + background-upload banner/refetch, AC 38–39/47–48)
- `web/src/components/experience/ReviewPhotoGrid.tsx` (new — UX polish, AC 44)
- `web/src/components/experience/ReviewPhotoLightbox.tsx` (new — UX polish, AC 45–46)
- `web/src/app/[locale]/resorts/[slug]/review/page.tsx`
- `web/src/app/[locale]/resorts/[slug]/review/ReviewPageClient.tsx`
- `web/messages/en.json` (UX polish — `addPhotos`, `photosUploading`, `lightbox.*`, updated hints; removed unused `emptyState` 2026-08-10)
- `web/e2e/review-text-crud.spec.ts` (text-only create/edit/delete E2E)
- `web/e2e/helpers/supabaseSession.ts` (Playwright cookie port default `3012`)
- `web/src/lib/analytics/phase2Events.ts`
- `docs/analytics/tracking-plan.md`
- `web/package.json` / `web/package-lock.json` (new dependency — `sharp`, UX polish AC 43)

### Change Log

- 2026-08-07: Story 12.2 created — submit/edit review with photos (ready-for-dev).
- 2026-08-07: Story 12.2 implemented — review write/edit/delete + photos; status → review.
- 2026-08-07: Post-review manual QA raised 7 UX issues; PO approved scope additions (AC 38–48, UX Polish Tasks, technical notes). Story remains `review` status — implementation of the polish items has **not** started, awaiting explicit go-ahead.
- 2026-08-07: UX Polish Tasks (AC 38–48) implemented — CTA consolidation, redesigned photo attach control, wider desktop modal, server-side photo resize (`sharp` + storage bucket cap migration 007), 2×2 photo grid + carousel lightbox on `ReviewCard`, and background/optimistic photo upload with settle-time analytics. Added unit tests (`clientForm`, `photoResize`, `backgroundPhotoUpload`, photo confirm route). `npm run lint && npm run build && npm run test:unit && npm run test:analytics` pass; `test:launch` still blocked only by the pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` gate. Status remains `review`, pending PO's UX-polish manual QA pass.
- 2026-08-10: PO sign-off — AC 38 amended (`countZero` only, no list `emptyState`); text-only E2E pass; Testing & DoD complete. Status → **done**.
