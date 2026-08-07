baseline_commit: db2638f

# Story 12.1: Experience Section Shell & Review List

Status: done

<!-- Ultimate context engine analysis completed - comprehensive developer guide created -->

## Story

As a **visitor on resort detail**,  
I want to see aggregate rating and community reviews,  
So that I trust peer experiences before my trip.

**Source shard(s):** [`epics/phase2/shards/epic-12-community-reviews.md`](../../planning-artifacts/epics/phase2/shards/epic-12-community-reviews.md) · [`prds/phase2/shards/06-features-reviews-comments.md`](../../planning-artifacts/prds/phase2/shards/06-features-reviews-comments.md) · UX [`ux-designs/ux-phase2/experience.md`](../../planning-artifacts/ux-designs/ux-phase2/experience.md) · [`design.md`](../../planning-artifacts/ux-designs/ux-phase2/design.md) §ExperienceSection, §ReviewCard · addendum [`§M`](../../planning-artifacts/prds/phase2/addendum.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Epic **9** complete (Supabase schema `reviews`, `review_photos`, `profiles`, `user_badges`; SSR clients). Epic **11** map section mounted above Experience in `ResortDetailContent`.

**Blocks:** Story **12.2** (submit/edit review + photos), **12.3** (comment thread), **12.4** (report + admin hide).

**Epic note:** Story **15.3** (community guidelines / legal) gates **public launch** of UGC — dev can ship behind feature flag or to staging before that story; do not block 12.1 implementation.

---

## PO decisions (defaults — epic silent)

| Topic | Decision |
|-------|----------|
| **Write a review CTA** | Include in 12.1 **shell only**: always visible per UX `ExperienceSection`. **Guest** → `openSignIn({ trigger: 'review', returnTo: current path })`. **Signed-in** → button renders but is **disabled** with `aria-disabled` until **12.2** wires the form (no dead-end navigation). |
| **Kebab menu (Edit / Report)** | **Out of scope 12.1** — owner Edit in **12.2**; Report in **12.4**. Read-only cards this story. |
| **Comments block** | **Out of scope 12.1** — flat thread in **12.3**. Do not render comment subheader or list yet. |
| **Review sort / pagination** | **Newest first** (`created_at desc`). Initial fetch **50** reviews; **Load more** button if total > 50 (matches comment UX cap in addendum §M). |
| **Badge chips on card** | Join `user_badges` for author; show up to **3** badge names as small caption chips using addendum §C definitions (`lib/badges/definitions.ts`). No badge evaluation logic (Epic 13). |
| **Aggregate display** | Average to **one decimal** (e.g. `4.3`); warm amber stars (`text-accent-warm` / design token). Count copy: `"23 reviews"` / `"1 review"` / empty state below. |
| **Zero reviews** | SSR aggregate shows **no star row** (or em dash) + count `"No reviews yet"`; client list shows empty state + Write CTA per UX (`experience.md` Reviews empty). |
| **Profile links** | Author name + avatar link to `/u/:username` even though **13.4** builds the page — href is correct; 404 until then is acceptable. |
| **JSON-LD rich snippets** | **Out of scope 12.1** — visible aggregate in SSR HTML satisfies FR-14.3; structured data deferred to **15.4** if needed. |
| **Dev seed data** | **Not required** — manual QA can insert rows via Supabase SQL or wait for **12.2**. Document sample INSERT in dev notes for PO. |

---

## Acceptance Criteria

### A. Section placement & shell

1. **Given** resort detail  
   **When** the page renders  
   **Then** a **"How people experienced it"** section appears **after** the map section (`ResortMapSection`) and **before** `PracticalTipsBlock` (addendum §M order #6)

2. **And** section uses `SectionHeader` with label copy **`How people experienced it`** (sentence case in UI; design.md badge style uppercase via component)

3. **And** section is **always rendered** for published resorts (even when review count is 0)

### B. SSR aggregate rating (FR-14.3)

4. **Given** any visitor (guest or signed-in)  
   **When** the initial HTML is delivered  
   **Then** aggregate **★ average** (one decimal when count > 0) and **review count** are present in **server-rendered HTML** (not client-only)

5. **And** aggregate is computed from `reviews` where `resort_slug = :slug` and `status = 'published'` (SQL `avg(rating)`, `count(*)`)

6. **And** RSC fetch uses **`revalidate: 60`** (architecture §6.6) — implement via `fetch(..., { next: { revalidate: 60 } })` on internal aggregate helper or `unstable_cache` with 60s TTL; do not client-fetch aggregate

7. **And** aggregate values are **not duplicated** inside individual review card HTML (review bodies are client-hydrated only per FR-14.3)

### C. Public GET reviews API

8. **Given** a client request  
   **When** `GET /api/reviews?resortSlug=niseko-united`  
   **Then** response is **200** with JSON envelope consistent with existing API patterns (`jsonOk`)

9. **And** returns only **`published`** reviews for that slug, ordered **`created_at desc`**

10. **And** each review includes: `id`, `rating`, `body`, `created_at`, `updated_at`, `author` (`username`, `display_name`, `avatar_url`), `badges` (array of `badge_id`), `photos` (array of `{ id, url, sort_order }` public URLs)

11. **And** route is **public** (no session required) — guests read without login wall

12. **And** supports optional `?limit=` (default **50**, max **50**) and `?offset=` for load-more (validate with Zod; return `{ reviews, total, hasMore }`)

13. **And** hidden/deleted reviews are **never** returned

### D. Client review list

14. **Given** the Experience section hydrated  
    **When** the client island mounts  
    **Then** it fetches `/api/reviews?resortSlug=` and renders a **skeleton** state until data arrives (UX `Reviews loading`)

15. **And** each review card shows:
    - Author **avatar** (40px, `rounded-avatar`) + **display name** linking to `/u/:username`
    - Up to **3** badge chips from author's earned badges
    - **Star row** for that review's 1–5 rating (warm amber)
    - **Body text** (sanitized output — stored text is plain; render with standard escaping)
    - **Relative date** (`updated_at` if edited after create, else `created_at`) as caption
    - **Photo strip** — horizontal scroll, max 5 thumbnails from `review-photos` bucket URLs when present

16. **And** **no login wall** intercepts reading the list

17. **Given** more than 50 published reviews  
    **When** the first page loads  
    **Then** show **Load more** to fetch next offset; append cards without full-page reload

18. **Given** zero reviews  
    **When** the list loads  
    **Then** show empty copy **"No reviews yet — be the first"** (UX) + Write CTA (AC A shell)

### E. Write review CTA (shell — form is 12.2)

19. **Given** a **guest**  
    **When** they tap **Write a review**  
    **Then** `SignInSheet` opens with `trigger: 'review'` and `returnTo` = current resort detail path

20. **Given** a **signed-in** user  
    **When** they view the CTA  
    **Then** button is visible but **disabled** until Story 12.2 (no broken navigation)

### F. Database / RLS prerequisite

21. **Given** published review photos exist  
    **When** anon/public client reads via API  
    **Then** photo rows are readable — add migration **`005_review_photos_public_read.sql`** with policy: SELECT on `review_photos` where parent review `status = 'published'` (current schema only allows owner SELECT per `004_account_deletion.sql`)

### G. i18n & accessibility

22. **And** all user-visible strings live under `messages/en.json` → `resortDetail.experience.*` (section title, count pluralization, empty state, load more, write CTA, relative dates if not handled by `Intl`)

23. **And** section has `aria-labelledby` pointing at section heading; star ratings expose accessible text (e.g. `"4 out of 5 stars"`)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** `lib/reviews/aggregate.ts` (avg/count, zero reviews); API query parser (limit/offset); photo public URL builder; badge chip cap helper
- [x] **API route test:** GET `/api/reviews` returns published only, hides `hidden`, includes author join shape (mock Supabase or handler unit test)
- [x] **Analytics:** N/A — no new events in 12.1 (`review_submitted` is **12.2**)
- [x] **Content / resorts:** `npm run test:launch` (section mount must not break detail SSG)
- [x] **Lint / build / unit:** `npm run lint && npm run build && npm run test:unit`
- [x] **E2E:** optional note — full review flow deferred to 12.2; manual PO path below
- [x] **Manual QA (PO):** resort with 0 reviews (empty + aggregate SSR); resort with seeded reviews (view source: aggregate in HTML, cards after hydration); guest reads without sign-in; guest Write → sign-in sheet; signed-in Write disabled; author link href `/u/:username`; photo strip scrolls

---

## Tasks / Subtasks

- [x] Migration `supabase/migrations/005_review_photos_public_read.sql` — public SELECT for photos on published reviews (AC: 21)
- [x] `web/src/lib/badges/definitions.ts` — read-only badge metadata from addendum §C (AC: 15)
- [x] `web/src/lib/reviews/` — `aggregate.ts`, `types.ts`, `photoUrl.ts`, `queries.ts` (AC: 4–7, 10)
- [x] `web/src/app/api/reviews/route.ts` — **GET only** in 12.1 (AC: 8–13)
- [x] `web/src/components/experience/` — `ExperienceSection.tsx` (RSC + props), `AggregateRating.tsx`, `ReviewList.tsx` (client), `ReviewCard.tsx`, `WriteReviewButton.tsx` (AC: 1–3, 14–20)
- [x] Wire into `ResortDetailContent.tsx` after map, before tips; pass `resortSlug` (AC: 1)
- [x] i18n keys `resortDetail.experience.*` (AC: 22)
- [x] Unit tests + verification commands (Testing section)

---

## Dev Notes

### Brownfield (post Epic 9 + 11)

| Area | Status |
|------|--------|
| `reviews` / `review_photos` / `profiles` / `user_badges` tables | ✅ `001_phase2_core.sql` |
| `review-photos` storage bucket public read | ✅ `002_storage_buckets.sql` |
| `review_photos` RLS SELECT for public | ❌ **Gap** — owner-only today; fix in 21 |
| `GET /api/reviews` | ❌ Not implemented |
| `web/src/components/experience/` | ❌ Not implemented |
| `ResortDetailContent` order | ✅ Map → (insert Experience) → Practical tips |
| `SignInTrigger` includes `'review'` | ✅ `lib/auth/types.ts` |
| Public profile route `/u/[username]` | ❌ Epic 13.4 — links OK anyway |

**Do not reinvent:** reuse `SectionHeader`, `jsonOk` / `createApiError`, `createClient()` from `@/lib/supabase/server`, `useAuth()` / `openSignIn` from `AuthProvider`, `Link` from `@/i18n/routing`, design tokens from Phase 2 `design.md`.

### Architecture compliance

- [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §3.1 dual storage · §5 `/api/reviews` GET public · §6.2 `components/experience/` · §6.6 SEO hybrid (aggregate RSC `revalidate: 60`, list client fetch)
- Resort detail remains **SSG/ISR** for editorial; Experience aggregate is a **small RSC sub-fetch**, not a move to full SSR page
- **No** `SUPABASE_SERVICE_ROLE_KEY` for public read paths — anon client + RLS is sufficient once photo policy fixed
- POST/PATCH review routes, sanitize, photo upload: **12.2** — do not scope-creep

### UX tokens (spines win)

| Token | Use |
|-------|-----|
| `typography.badge` | Section label styling via `SectionHeader` |
| `typography.section-heading` | Aggregate number |
| `colors.star-filled` / `accent-warm` | Stars |
| `rounded.card`, `colors.surface` | ReviewCard |
| `rounded.avatar` 40px | Author row |
| `review-photo-gap` 8px | Horizontal photo strip |

Reference mock: [`mockups/key-resort-detail-phase2.html`](../../planning-artifacts/ux-designs/ux-phase2/mockups/key-resort-detail-phase2.html)

### API response shape (suggested)

```typescript
type ReviewAuthor = {
  username: string;
  display_name: string;
  avatar_url: string | null;
};

type ReviewPhoto = {
  id: string;
  url: string;
  sort_order: number;
};

type PublicReview = {
  id: string;
  rating: number;
  body: string;
  created_at: string;
  updated_at: string;
  author: ReviewAuthor;
  badges: string[]; // badge_id[]
  photos: ReviewPhoto[];
};

// GET /api/reviews?resortSlug=&limit=50&offset=0
type ReviewsListResponse = {
  reviews: PublicReview[];
  total: number;
  hasMore: boolean;
};
```

Photo URL pattern: `{NEXT_PUBLIC_SUPABASE_URL}/storage/v1/object/public/review-photos/{storage_path}` — mirror avatar URL helper in `lib/auth/avatarUrl.ts`.

### Aggregate SSR pattern (suggested)

```typescript
// lib/reviews/aggregate.ts — called from ExperienceSection (RSC)
export async function getReviewAggregate(resortSlug: string): Promise<{
  average: number | null; // null when count === 0
  count: number;
}> {
  const supabase = await createClient();
  const { data, error } = await supabase
    .from('reviews')
    .select('rating')
    .eq('resort_slug', resortSlug)
    .eq('status', 'published');
  // compute avg + count; return { average: null, count: 0 } on empty
}
```

Wrap page section in `<ExperienceSection resortSlug={...} aggregate={...} />` where aggregate is fetched in RSC parent (`ExperienceSection` async server component) with `export const revalidate = 60` on the segment **or** cached helper — pick one pattern and document in PR.

### Manual QA seed (optional SQL)

```sql
-- After a test user exists with profile; replace UUIDs/slug
insert into public.reviews (user_id, resort_slug, rating, body)
values ('<user-uuid>', 'myoko-kogen', 5, 'Quiet trees on a powder day — worth the trip from Tokyo.');
```

### Epic 11 retro carry-forward

- **`ResortDetailContent` mount point** — map already between Getting There and Practical tips; insert Experience **immediately after** `ResortMapSection` (even when map hidden due to missing geo — Experience is independent of lat/lng)
- **Cross-breakpoint QA** — verify section spacing matches `gap-6` / `SectionHeader` rhythm from map section
- **No map E2E precedent** — manual PO QA acceptable for first UGC read path

### Out of scope (explicit)

- Review submit/edit, photo upload, sanitize writes (**12.2**)
- Comment thread (**12.3**)
- Report button, admin hide, moderation (**12.4**)
- Badge evaluation on publish (**13.2**)
- Public profile page (**13.4**)
- Analytics `review_submitted` (**12.2**)
- Community guidelines legal copy (**15.3**)

---

## Dev Agent Record

### Agent Model Used

Composer (Cursor)

### Debug Log References

- PO clarifications: map-less mount (A), zero aggregate without stars (A), half-star aggregate display, badge cap by `earned_at desc`, disabled Write CTA only, no feature flag, `unstable_cache` revalidate 60, edited label (B), initials avatar fallback.

### Completion Notes List

- Experience section mounts after map slot (or Getting There when no geo), always visible on published resort detail.
- SSR aggregate via `getReviewAggregate` + `unstable_cache` (60s); client list hydrates from public `GET /api/reviews`.
- Half-star aggregate row (rounded to nearest 0.5); integer star rows on review cards.
- Migration `005_review_photos_public_read.sql` adds anon SELECT on `review_photos` for published reviews — **apply on Supabase before PO manual QA with photos**.
- Verification: `npm run lint`, `npm run build`, `npm run test:unit` (293 tests) pass. `test:launch` exits 1 due to pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env gate (unchanged by this story); build/SSG for all 20 resorts succeeds.

### File List

- `supabase/migrations/005_review_photos_public_read.sql`
- `web/src/lib/badges/definitions.ts`
- `web/src/lib/reviews/aggregate.ts`
- `web/src/lib/reviews/aggregate.test.ts`
- `web/src/lib/reviews/badgeChips.ts`
- `web/src/lib/reviews/badgeChips.test.ts`
- `web/src/lib/reviews/formatReviewDate.ts`
- `web/src/lib/reviews/formatReviewDate.test.ts`
- `web/src/lib/reviews/photoUrl.ts`
- `web/src/lib/reviews/photoUrl.test.ts`
- `web/src/lib/reviews/queries.ts`
- `web/src/lib/reviews/queries.test.ts`
- `web/src/lib/reviews/queryParams.ts`
- `web/src/lib/reviews/queryParams.test.ts`
- `web/src/lib/reviews/starRating.ts`
- `web/src/lib/reviews/starRating.test.ts`
- `web/src/lib/reviews/types.ts`
- `web/src/app/api/reviews/route.ts`
- `web/src/app/api/reviews/route.test.ts`
- `web/src/components/experience/AggregateRating.tsx`
- `web/src/components/experience/ExperienceSection.tsx`
- `web/src/components/experience/ReviewCard.tsx`
- `web/src/components/experience/ReviewList.tsx`
- `web/src/components/experience/StarRating.tsx`
- `web/src/components/experience/WriteReviewButton.tsx`
- `web/src/components/resort/ResortDetailContent.tsx`
- `web/src/components/ui/SectionHeader.tsx`
- `web/messages/en.json`

### Change Log

- 2026-08-07: Story 12.1 — Experience section shell, SSR aggregate, public reviews API, client review list (review).
- 2026-08-07: PO manual QA sign-off — empty state, SSR aggregate, guest read, sign-in CTA, disabled Write for signed-in, author links, photo strip (done).
