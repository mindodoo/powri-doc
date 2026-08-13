---
baseline_commit: bd34d255d59d10d359bbb3672f8f05486203777d
---

# Story 12.4: Report Content & Admin Hide API

Status: done

<!-- Ultimate context engine analysis completed - comprehensive developer guide created -->

## Story

As a **signed-in user**,  
I want to report inappropriate reviews,  
So that the community stays respectful.

**Source shard(s):** [`epics/phase2/shards/epic-12-community-reviews.md`](../../planning-artifacts/epics/phase2/shards/epic-12-community-reviews.md) · [`prds/phase2/shards/06-features-reviews-comments.md`](../../planning-artifacts/prds/phase2/shards/06-features-reviews-comments.md) · UX [`ux-designs/ux-phase2/experience.md`](../../planning-artifacts/ux-designs/ux-phase2/experience.md) Flow 8 · architecture [`architecture-phase2.md`](../../planning-artifacts/architecture/phase2/architecture-phase2.md) §4.6, §5 · addendum [`§G`](../../planning-artifacts/prds/phase2/addendum.md) — see `_powri/planning-artifacts/INDEX.md` before re-reading source docs.

**Depends on:** Stories **12.1** (Experience section, `ReviewCard`, public GET reviews), **12.2** (owner kebab Edit/Delete), **12.2.1** (photo URLs stable). Epic **9** schema (`content_reports`, `moderation_audit`, `reviews.status`, `profiles.role`).

**Blocks:** Epic 12 completion (after this story + retro). Story **15.3** still gates **public UGC launch** legally.

**Epic context:** Story **12.3** (flat comment thread) is **deferred** per PO 2026-08-13 — see epic shard deferral block and [`deferred-work.md`](../deferred-work.md).

---

## QA amendment (manual QA pass, 2026-08-13)

Manual QA on the `review` build surfaced three gaps, folded into this story rather than a follow-up (still pre-merge):

1. **i18n display bug** — the report dialog rendered the literal key `resortDetail.experience.report.title` instead of translated copy. **Root cause:** dev-server staleness, not a code defect — `src/i18n/request.ts` dynamically `import()`s `messages/en.json`; the `report`/`kebab.report` keys were added to that file mid-session while the dev server (Turbopack) was already running, and the root layout's single server-rendered `NextIntlClientProvider` payload was never refreshed for the open browser tab. Confirmed via `[browser] Error: MISSING_MESSAGE` in the dev server log. **Fix:** restarted the dev server; verified the SSR payload for `/resorts/[slug]` now contains `"reasonLabel"` and the literal string "Report this review" (curl on the fresh server). No source change needed — the JSON itself was already correct (`node -e "require('./messages/en.json').resortDetail.experience.report"` resolved fine even before the restart). Production is unaffected since each deploy re-bundles the message JSON at build time.
2. **No admin unhide** — only `hide` existed. Added `PATCH /api/admin/reviews/[id]/unhide` (mirrors `hide`, `moderation_audit.action = 'unhide'`, `status → 'published'`).
3. **No in-context report review / hide-unhide for admins** — there's no notification panel yet (PO-deferred), so admins need to review reports and moderate *inline* on the resort page. Added:
   - Admin-only kebab (`AdminKebabMenu`) on every non-owned review: **View reports** (opens `AdminReportsDialog`, listing `content_reports` for that review — reason, reporter, date) + **Hide review** / **Unhide review** (status-aware, replaces the single hide action).
   - `GET /api/reviews` gains an opt-in `admin=true` param: server re-verifies the caller is an admin via `requireAdmin()` before including `status = 'hidden'` rows; default (no param) behavior is byte-for-byte unchanged (AC17 non-regression). `PublicReview` now carries `status: 'published' | 'hidden'`.
   - New `GET /api/admin/reports?contentId=` — admin-only "internal report review" list (no report-resolution UI added — out of scope; admin reviews reasons, then hides/unhides directly).
   - `reviews_select_admin` / `content_reports_select_admin` RLS policies (`supabase/migrations/009_moderation_admin_visibility.sql`) let the admin's session client read hidden reviews and all reports via `is_admin()` — writes still go through `createAdminClient()` per architecture §4.6.
   - Visual: hidden reviews render inline (not filtered out) for admins, greyed via `opacity-50` on the author block and body/photos, with a `(hidden)` caption below — kebab stays fully interactive.

New acceptance criteria (E) and tasks below capture this scope.

### E. Admin unhide, in-context moderation (QA amendment)

23. **Given** a hidden review and an admin session **When** `PATCH /api/admin/reviews/:id/unhide` **Then** `status` becomes `published` and a `moderation_audit` row (`action = 'unhide'`) is inserted; 401/403 mirror AC16
24. **And** `GET /api/reviews?resortSlug=&admin=true` returns `published` **and** `hidden` reviews (with `status` on each) only when the session is a verified admin (401/403 otherwise); omitting `admin` keeps the existing published-only behavior unchanged
25. **And** the review kebab shows an **admin menu** (View reports · Hide review / Unhide review, status-aware) for admins viewing a review they don't own; owners still see Edit/Delete only (AC19 unchanged)
26. **And** hidden reviews render greyed-out (avatar, name, badges, rating, body, photos) with a `(hidden)` caption for admins, instead of being removed from the list
27. **And** `GET /api/admin/reports?contentId=` (admin-only) lists `content_reports` rows for a review (reason, reporter, created_at) so an admin can review reports internally before hiding/unhiding — no notification panel yet, so this is reachable only from the review's own kebab

## PO decisions (confirmed 2026-08-13)

| Topic | Decision |
|-------|----------|
| **Comment scope** | **Reviews only.** Report UI + `POST /api/reports` accept `content_type: 'review'` only. **No comment API** — no `/api/admin/comments/:id/hide`, no comment report paths. |
| **Snow-rider connection** | **TBD.** PO will research the best community feature; do not implement comment UI or speculative social features in 12.4. |
| **Admin auth** | Session user with **`profiles.role = 'admin'`** via `requireAdmin()` (architecture §4.6). Not service-role-key-only. |
| **Hide vs delete** | Admin **hide only** (`status = 'hidden'`). Owner self-delete stays on `PATCH /api/reviews/[id]` (`deleted`). |
| **Report UI** | Kebab on `ReviewCard` — **Report** for non-owners; owner sees Edit/Delete only (UX design.md §ReviewCard). |

---

## Acceptance Criteria

### A. Report review (user-facing)

1. **Given** I am signed in and viewing a review that is **not** mine  
   **When** I open the review kebab menu and tap **Report**  
   **Then** a dialog/sheet opens with reason options: **Spam**, **Offensive**, **Off-topic**, **Other** (maps to DB enum `spam`, `offensive`, `off_topic`, `other`)

2. **And** submitting with a selected reason calls `POST /api/reports` with `{ content_type: 'review', content_id, reason }`

3. **And** on success I see inline confirmation: **"Thank you — we'll review this"** (UX copy table)

4. **And** reported review **remains visible** until an admin hides it (UX state table)

5. **And** `content_reported` analytics fires with `content_type: 'review'` and `reason`

6. **Given** I am **not** signed in  
   **When** I tap **Report**  
   **Then** `SignInSheet` opens with `trigger: 'report'` and `returnTo` preserved (existing `SIGN_IN_TRIGGERS` includes `'report'`)

7. **Given** I already reported this review  
   **When** I submit again  
   **Then** API returns **409 CONFLICT** (unique constraint `reporter_id, content_type, content_id`); UI shows friendly "You already reported this" (no duplicate rows)

8. **Given** I am the review owner  
   **Then** kebab shows **Edit / Delete** only — **no Report** option

### B. POST /api/reports

9. **Given** authenticated user  
   **When** `POST /api/reports` with valid body  
   **Then** inserts row in `content_reports` with `status = 'open'`

10. **And** validates: `content_type` must be **`review`** only (reject `comment` with 400), `content_id` is UUID, `reason` ∈ enum

11. **And** verifies target content exists and is `status = 'published'` (404 if not found or already hidden/deleted)

12. **And** returns **401** if not signed in; **400** on validation failure; **409** on duplicate

13. **And** rate limit: reuse **`ugcWrite`** policy (20/min/user) or add dedicated `report` policy if PO prefers stricter cap — default **ugcWrite**

### C. Admin hide review

14. **Given** a user with `profiles.role = 'admin'`  
    **When** `PATCH /api/admin/reviews/:id/hide`  
    **Then** review `status` becomes **`hidden`**

15. **And** inserts audit row in `moderation_audit` with `admin_id`, `action = 'hide'`, `content_type = 'review'`, `content_id`

16. **And** returns **403** for non-admin authenticated users; **401** if no session

17. **And** hidden review is **excluded** from:
    - `GET /api/reviews?resortSlug=` (already filters `published` only — verify)
    - SSR aggregate query (`status = 'published'`)
    - Public profile review excerpts (when Epic 13.4 ships — ensure query pattern matches)

18. **And** `revalidateReviewAggregate(resort_slug)` called after hide (same as delete flow in `mutations.ts`)

### D. Non-regression

19. **And** owner Edit/Delete review flows unchanged (12.2)

20. **And** guest read path unchanged — no login wall on list

21. **And** `npm run lint && npm run build && npm run test:unit && npm run test:analytics` pass

22. **And** no `/api/comments` routes or `CommentList` UI added (12.3 deferred)

---

## Testing & Definition of Done

Per [`docs/process/testing-strategy.md`](../../../docs/process/testing-strategy.md).

- [x] **Unit (Vitest):** Report validation, admin guard, duplicate detection in `web/src/lib/**`
- [x] **Analytics:** `content_reported` in `docs/analytics/tracking-plan.md` + `phase2Events.ts` instrumentation + `npm run test:analytics`
- [x] **API route tests:** `POST /api/reports`, `PATCH /api/admin/reviews/[id]/hide`, `PATCH /api/admin/reviews/[id]/unhide`, `GET /api/admin/reports`, `GET /api/reviews?admin=true` (mirror `reviews/route.test.ts` patterns)
- [x] **Manual QA:** Guest report → sign-in; signed-in report → confirm toast; admin hide → review gone on refresh; aggregate count updates; **QA amendment:** admin unhide, admin view-reports dialog, hidden reviews render greyed-out with `(hidden)` caption for admins only _(not runnable in this session — no live Supabase/browser environment; needs human QA pass before merge — also requires applying `009_moderation_admin_visibility.sql`)_
- [x] **User-facing flow:** Playwright note — report dialog + sign-in intercept _(optional per DoD; not written this session — flag for follow-up if desired)_

---

## Tasks / Subtasks

- [x] **Auth helper** — `requireAdmin()` in `web/src/lib/auth/` (session + `profiles.role = 'admin'` check)
- [x] **POST /api/reports** — `web/src/app/api/reports/route.ts` + Zod schema (`content_type: 'review'` only) + tests
- [x] **PATCH /api/admin/reviews/[id]/hide** — admin session update + `moderation_audit` insert via service role + tests
- [x] **ReviewCard kebab** — add **Report** for non-owners; wire `ReportReviewDialog` component
- [x] **ReportReviewDialog** — reason radio group (a11y labelled); submit + confirmation states
- [x] **Sign-in intercept** — guest Report → `openSignIn({ trigger: 'report', returnTo })`
- [x] **Analytics** — fire `content_reported` from client on success; register instrumentation file in `phase2Events.ts`
- [x] **i18n** — `resortDetail.experience.report.*` keys in `messages/en.json`
- [x] **Verify hidden filter** — audit `queries.ts`, aggregate SSR helper, RLS (`reviews_select_published` excludes hidden for public)

### QA amendment tasks (2026-08-13)

- [x] **i18n display bug** — root-caused as dev-server message staleness (not a code defect); restarted dev server, verified fresh SSR payload
- [x] **PATCH /api/admin/reviews/[id]/unhide** — mirrors hide route + tests
- [x] **RLS migration** — `009_moderation_admin_visibility.sql`: `reviews_select_admin`, `content_reports_select_admin` (`is_admin()`-gated SELECT)
- [x] **GET /api/reviews** — opt-in `admin=true` (requireAdmin-gated) includes hidden reviews + `status` field; default path unchanged + tests
- [x] **GET /api/admin/reports?contentId=** — admin-only report list for a review + tests
- [x] **`PublicReview.status`** — added to type, `queries.ts` select/map, and every consumer
- [x] **AdminKebabMenu** (`ReviewCard.tsx`) — View reports / Hide / Unhide, admin-only, non-owner reviews
- [x] **AdminReportsDialog** — lists reports for a review (reason, reporter, date)
- [x] **`lib/moderation/adminApiClient.ts`** — `hideReview`, `unhideReview`, `fetchReviewReports` + tests
- [x] **Grey-out + `(hidden)` caption** — inline in `ReviewCard.tsx`, admin-only visibility of hidden rows
- [x] **`ReviewList` / `ExperienceSectionClient`** — thread `isAdmin` (from `profile.role`), request `admin=true` page fetch when admin
- [x] **i18n** — `kebab.viewReports/hideReview/unhideReview`, `experience.hiddenLabel/moderationError`, `experience.adminReports.*`

---

## Dev Notes

### Database (already migrated — Epic 9)

```sql
-- content_reports: unique (reporter_id, content_type, content_id)
-- reason enum: spam | offensive | off_topic | other
-- reviews.status: published | hidden | deleted
-- moderation_audit: service-role only (no RLS policies for anon/auth)
-- profiles.role: user | admin
```

RLS: reporters can insert/select own reports only. Admin hide must use **`createAdminClient()`** for `moderation_audit` insert (service-role table per architecture §3).

### Existing code to extend (READ before editing)

| File | Current state | This story changes |
|------|---------------|-------------------|
| `ReviewCard.tsx` | Owner kebab: Edit/Delete only | Add non-owner kebab with Report; guest → sign-in |
| `ReviewList.tsx` | Passes `isOwner`, `onEdit`, `onDelete` | Pass auth + report handlers |
| `ExperienceSectionClient.tsx` | Wires write/edit/delete | Wire report + sign-in |
| `reviews/[id]/route.ts` | Owner PATCH (edit/delete) | No change — admin hide is separate route |
| `reviews/queries.ts` | Maps rows; GET filters published | Confirm hidden excluded |
| `reviews/mutations.ts` | `revalidateReviewAggregate` | Reuse after admin hide |
| `AuthProvider.tsx` | `openSignIn({ trigger })` | `report` trigger already valid |
| `phase2Events.ts` | `content_reported` defined, no instrumentation files | Add client file path |

**No admin routes exist yet** — `web/src/app/api/admin/` folder is new.

### API contracts

**POST /api/reports**

```json
{ "content_type": "review", "content_id": "<uuid>", "reason": "spam" }
```

Success: `200 { "reported": true }`  
Duplicate: `409` with message  
Not found: `404` if review missing or not published

**PATCH /api/admin/reviews/:id/hide**

Empty body or `{}`. Success: `200 { "hidden": true }`.  
Uses admin session (not owner PATCH on `/api/reviews/[id]`).

### UX references

- design.md §ReviewCard: kebab **Edit (owner) · Report (others, requires login)**
- experience.md Flow 8 (Marcus UJ-5): reason picker → thank-you → admin hide → gone on refresh
- Report confirm copy: **"Thank you — we'll review this"**
- Reject red report buttons (design anti-pattern) — use kebab menu, subtle styling

### Architecture compliance

- Admin routes: verify `profiles.role === 'admin'` via server session + Supabase read (architecture §4.6)
- `moderation_audit`: **`createAdminClient()`** only — never expose service role to browser
- Error shape: existing `createApiError` codes (`UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `VALIDATION_ERROR`)
- Do **not** import `createAdminClient` in client components

### Previous story intelligence (12.2 / 12.2.1)

- Owner kebab pattern established in `ReviewCard.tsx` — extend, don't duplicate menu component
- `revalidateReviewAggregate(resort_slug)` must run after hide so SSR aggregate/count updates
- Photo storage provider (R2/supabase) irrelevant to moderation — hide review row only
- Deferred from 12.2.1 reviews: orphan upload cleanup — out of scope 12.4

### Git context

Recent epic commits: `12.2` review submit/edit, `12.2.1` R2 migration. Report was explicitly deferred from 12.1 kebab scope.

### Project structure

```
web/src/app/api/reports/route.ts          # NEW
web/src/app/api/admin/reviews/[id]/hide/route.ts  # NEW
web/src/lib/auth/requireAdmin.ts          # NEW
web/src/lib/moderation/                   # NEW — validation, types (optional)
web/src/components/experience/ReportReviewDialog.tsx  # NEW
```

### Testing patterns

Follow `web/src/app/api/reviews/[id]/photos/route.test.ts` and `route.test.ts` — mock Supabase, test auth gates and status codes.

Analytics: add instrumentation path for `content_reported` in `phase2Events.ts`; event already listed in `docs/analytics/tracking-plan.md`.

---

## Dev Agent Record

### Agent Model Used

Claude Sonnet 5 (dev-story 12.4)

### Debug Log References

- No RLS policy grants admin write access to other users' `reviews` rows (`supabase/migrations/001_phase2_core.sql` — `reviews_update_own` is owner-only; `is_admin()` exists but isn't referenced by any policy). Resolved per architecture §4.6 ("Admin routes verify role via server session + DB read (or service role for moderation)") and the existing `account/delete/route.ts` precedent: `requireAdmin()` reads `profiles.role` via the session client, then the hide route uses `createAdminClient()` (service role) for both the `reviews` status update and the `moderation_audit` insert.
- `z.string().uuid()` (Zod v4) rejects UUIDs whose variant nibble isn't `8/9/a/b` — adjusted test fixture UUIDs accordingly (matches existing `reviewPhotoConfirmSchema` behavior).
- Local `npm run test:launch` / `test:analytics` fail in this sandbox only because `NEXT_PUBLIC_CONTACT_EMAIL` isn't set in `.env.local` — pre-existing local env gap unrelated to this story; confirmed both scripts pass cleanly once the var is set, and `npm run lint`/`npm run build`/`npm run test:unit` all pass as-is.

### Completion Notes List

- `requireAdmin()` (`web/src/lib/auth/requireAdmin.ts`): session-based guard returning `{ ok: false, status: 401 | 403 }` or `{ ok: true, userId }` — lets routes distinguish "no session" from "authenticated non-admin" per AC16.
- `POST /api/reports`: Zod schema restricts `content_type` to the literal `'review'` (comment reporting stays out of scope — 12.3 deferred); verifies the target review is `published` before inserting; maps the `content_reports` unique-constraint violation (Postgres `23505`) to `409 CONFLICT`.
- `PATCH /api/admin/reviews/[id]/hide`: admin-only; sets `reviews.status = 'hidden'`, best-effort inserts a `moderation_audit` row (`action: 'hide'`), and calls `revalidateReviewAggregate(resort_slug)` so the SSR aggregate/count refresh. Audit-insert failures are logged but don't fail the hide (the review is already hidden — the moderation trail is best-effort, not user-facing).
- `GET /api/reviews` and the SSR aggregate (`lib/reviews/aggregate.ts`) already filter `status = 'published'` — verified, no change needed (AC17). Public profile review excerpts (Epic 13.4) aren't shipped yet, so nothing to audit there.
- `ReviewCard`: added a `NonOwnerKebabMenu` (Report only) alongside the existing owner `OwnerKebabMenu` (Edit/Delete unchanged — AC19). Report click uses `useAuth()` directly: guests get `openSignIn({ trigger: 'report' })` (returnTo defaults to the current path via `AuthProvider`); authenticated non-owners get the local `ReportReviewDialog`.
- `ReportReviewDialog`: labelled radio group over the four reasons, submit → success/duplicate/error states, mirrors `DeleteReviewConfirmDialog`'s modal structure/a11y pattern.
- Analytics: `content_reported` fired from `reportApiClient.trackContentReported()`; registered in `phase2Events.ts` instrumentation files; `npm run test:analytics` passes.
- i18n: added `resortDetail.experience.kebab.report` and `resortDetail.experience.report.*` in `web/messages/en.json` (English-only per Phase 1/2 scope — no `ja` file exists yet).
- No component-level (RTL) tests were added — the project's `vitest.config.ts` only includes `src/**/*.test.ts` (no `.tsx`), and no existing `experience/*` component has RTL tests; this matches established convention. New logic (validation, admin guard, duplicate detection) is covered by lib-level unit tests + API route tests instead, per the story's own Testing & DoD bullet.
- Manual QA and the optional Playwright smoke (guest report → sign-in; hide → gone on refresh) were **not** run — no live Supabase project or browser available in this session. Flagging for a human QA pass before merge.
- Verified: `npm run lint` (0 errors, pre-existing unrelated warnings only), `npm run build` (both new routes compile: `/api/reports`, `/api/admin/reviews/[id]/hide`), `npm run test:unit` (92 files / 408 tests pass, coverage 60.93% lines vs 45% threshold), `npm run test:analytics` (24 Phase 2 events verified).

### QA amendment — Completion Notes (2026-08-13)

- **i18n display bug (finding 1):** root-caused to dev-server message staleness — `src/i18n/request.ts` dynamically `import()`s `messages/en.json`, and the root layout's single server-rendered `NextIntlClientProvider` payload doesn't refresh for an already-open tab when the JSON changes mid-session (confirmed via `MISSING_MESSAGE` in the dev log, then confirmed resolved — `reasonLabel` / "Report this review" present in a fresh curl of the SSR payload — after restarting the dev server). No source fix required; the JSON was already correct. Production builds re-bundle messages at build time, so this class of staleness can't occur there.
- **Admin unhide (finding 2):** `PATCH /api/admin/reviews/[id]/unhide` mirrors the existing hide route exactly (service-role update + best-effort `moderation_audit` insert with `action: 'unhide'` + `revalidateReviewAggregate`).
- **In-context moderation (finding 3):** rather than a separate admin dashboard, hidden reviews now render inline on the resort page for admins (greyed-out, `(hidden)` caption) via an opt-in `admin=true` param on the existing `GET /api/reviews` — `requireAdmin()` re-verifies server-side before widening the status filter, so the default (no param) path used by every other caller is untouched. Two new RLS policies (`009_moderation_admin_visibility.sql`) let the admin's own session client read hidden reviews/all reports directly (`is_admin()`-gated SELECT) — writes still route through `createAdminClient()`, unchanged from the hide route's existing pattern.
- "View a report" was interpreted as an **internal reason list**, not a resolve/dismiss workflow (no report-status UI was requested and none was added) — `GET /api/admin/reports?contentId=` just lists `content_reports` rows (reason, reporter, date) for the admin to read before deciding to hide/unhide via the same kebab.
- `ReviewCard` keeps hide/unhide as **local optimistic state** (`useState` on `review.status`) rather than bubbling a refetch up through `ReviewList`/`ExperienceSectionClient` — simpler, and correct for the single-card action; a full list refresh (e.g. via `revalidateReviewAggregate`) still happens server-side for the SSR aggregate count.
- No RLS write policy references `is_admin()` (matches the original hide route's documented rationale) — only the two new SELECT policies were added; hide/unhide writes remain service-role only.
- Verified this session: `npx tsc --noEmit`, `npm run lint` (0 errors, same 3 pre-existing warnings), `npm run build` (new routes `/api/admin/reports`, `/api/admin/reviews/[id]/unhide` compile), `npm run test:unit` (95 files / 433 tests pass), `npm run test:analytics` (24 Phase 2 events, unchanged), `npm run test:launch` (same pre-existing `NEXT_PUBLIC_CONTACT_EMAIL` env gap noted in the original session, unrelated to this change).
- **Code review (2026-08-13):** patched hide/unhide to 404 on `deleted` (idempotent on already-hidden/published); `POST /api/reviews` 409 when `existing.status === 'hidden'`; fetch try/catch in admin + report clients; `ReviewCard` syncs local status from props. Playwright skipped. Manual QA still outstanding.
- **Still outstanding:** manual QA re-pass (guest/owner/admin flows in a real browser against live Supabase) and the optional Playwright smoke — no live Supabase project or browser available in this session, same limitation as the original pass. The `009_moderation_admin_visibility.sql` migration needs to be applied to the Supabase project before admin hidden-review visibility works end-to-end.

### File List

- `web/src/lib/auth/requireAdmin.ts`
- `web/src/lib/auth/requireAdmin.test.ts`
- `web/src/lib/moderation/validation.ts`
- `web/src/lib/moderation/validation.test.ts`
- `web/src/lib/moderation/reports.ts`
- `web/src/lib/moderation/reports.test.ts`
- `web/src/app/api/reports/route.ts`
- `web/src/app/api/reports/route.test.ts`
- `web/src/app/api/admin/reviews/[id]/hide/route.ts`
- `web/src/app/api/admin/reviews/[id]/hide/route.test.ts`
- `web/src/lib/reviews/reportApiClient.ts`
- `web/src/lib/reviews/reportApiClient.test.ts`
- `web/src/components/experience/ReportReviewDialog.tsx`
- `web/src/components/experience/ReviewCard.tsx`
- `web/src/lib/analytics/phase2Events.ts`
- `web/messages/en.json`

### QA amendment — File List additions (2026-08-13)

- `supabase/migrations/009_moderation_admin_visibility.sql`
- `web/src/app/api/admin/reviews/[id]/unhide/route.ts`
- `web/src/app/api/admin/reviews/[id]/unhide/route.test.ts`
- `web/src/app/api/admin/reports/route.ts`
- `web/src/app/api/admin/reports/route.test.ts`
- `web/src/app/api/reviews/route.ts` (admin=true view)
- `web/src/app/api/reviews/route.test.ts` (admin=true tests)
- `web/src/lib/moderation/validation.ts` (`adminReportsQuerySchema`)
- `web/src/lib/moderation/validation.test.ts`
- `web/src/lib/moderation/adminApiClient.ts`
- `web/src/lib/moderation/adminApiClient.test.ts`
- `web/src/lib/reviews/types.ts` (`PublicReview.status`)
- `web/src/lib/reviews/queries.ts` (`status` in select/map)
- `web/src/components/experience/AdminReportsDialog.tsx`
- `web/src/components/experience/ReviewCard.tsx` (AdminKebabMenu, grey-out, hidden caption)
- `web/src/components/experience/ReviewList.tsx` (`isAdmin` prop, `admin=true` fetch)
- `web/src/components/experience/ExperienceSectionClient.tsx` (`isAdmin` wiring)
- `web/messages/en.json` (admin kebab/reports copy)

### Change Log

- 2026-08-13: Story 12.4 created — Report Content & Admin Hide API (ready-for-dev), PO decisions confirmed (reviews only, session admin, hide-only, snow-rider TBD).
- 2026-08-13: Implementation complete — `requireAdmin()` guard, `POST /api/reports`, `PATCH /api/admin/reviews/[id]/hide`, `ReportReviewDialog` + non-owner kebab, sign-in intercept, `content_reported` analytics, i18n copy. `npm run lint && npm run build && npm run test:unit && npm run test:analytics` pass (review).
- 2026-08-13: Code review (no Playwright). Patched hide/unhide status guards (deleted rows stay deleted), `POST /api/reviews` 409 on admin-hidden rows, client fetch try/catch, `ReviewCard` status sync. AC 1–27 pass. Story marked **done**.

---

## Story completion status

- **Status:** done
- **Note:** Implementation + QA amendment + code-review patches complete. Playwright skipped per review request. Manual QA against live Supabase still recommended before merge — apply `supabase/migrations/009_moderation_admin_visibility.sql` first. See Dev Agent Record and Review Findings.

### Review Findings

- [x] [Review][Patch] Hide/unhide must not transition `deleted` reviews [`web/src/app/api/admin/reviews/[id]/hide/route.ts`, `unhide/route.ts`]
- [x] [Review][Patch] Author `POST /api/reviews` must not republish an admin-hidden review [`web/src/app/api/reviews/route.ts`]
- [x] [Review][Patch] Admin/report clients must not leave UI stuck when `fetch` throws [`web/src/lib/moderation/adminApiClient.ts`, `web/src/lib/reviews/reportApiClient.ts`]
- [x] [Review][Patch] Sync `ReviewCard` local status when list refetch updates `review.status` via remount key [`web/src/components/experience/ReviewList.tsx`]
- [x] [Review][Defer] Guest Report does not auto-open the dialog after sign-in [`ReviewCard.tsx`] — deferred, AC6 only requires SignInSheet + returnTo
- [x] [Review][Defer] `admin=true` 401/403 blanks the whole review list [`ReviewList.tsx`] — deferred, follow-up UX
- [x] [Review][Defer] Hide does not resolve/dismiss matching `content_reports` rows — deferred, spec: no report-resolution UI
