---
baseline_commit: 541ece0e496a4b0c654bac055900bbb610223924
---

# Bug 9.9: Magic link sending — AUTH_ERROR on all production attempts; misleading error copy

Status: done

**Epic:** 9 · **Shard:** [`epic-09-supabase-auth.md`](../planning-artifacts/epics/phase2/shards/epic-09-supabase-auth.md)  
**Reported:** 2026-07-03 · **Environment:** Production (`powri.vercel.app`)  
**Severity:** P1 (delivery) — **in-scope fix:** misleading error copy only (see Scope decision)  
**Blocks:** Bug 9.8 production re-verification remains blocked until magic-link delivery is unblocked (separate future story/ops)

---

## Scope decision (PO, 2026-07-03)

**In scope (done):** App-side error code mapping and user-facing copy — no misleading “check your email” when delivery failed; distinct copy for rate-limit responses.

**Out of scope / deferred:**

- Custom SMTP (Resend or other provider) on `powri-prod`
- Production magic-link delivery retest (AC 1, 2)
- Bug 9.8 callback re-verification via magic link (AC 6)

**Rationale:** Supabase built-in email is capped at **2 emails/hour per project**, which blocks proactive QA and reproduction on production. Powri currently uses **Vercel** (`powri.vercel.app`) with **no owned sending domain** — Resend/custom SMTP is not pursued at this time. Google OAuth remains the viable production sign-in path until a domain + SMTP story is scheduled.

Reference runbook [`docs/ops/supabase-prod-email-smtp.md`](../../docs/ops/supabase-prod-email-smtp.md) is retained for a future ops story; not executed for 9.9.

---

## Symptom

On production (`powri.vercel.app`):

1. **Any** attempt to send a magic link fails — both new sign-up (first email attempt) and existing account re-sign-in
2. The UI shows: **"Could not send the sign-in link. Check your email and try again."** *(fixed in app code — see Completion Notes)*
3. The API response contains error code **`AUTH_ERROR`** *(fixed — mapped codes replace `AUTH_ERROR`)*
4. No email is delivered to the user *(unchanged — Supabase rate limit; deferred)*
5. This affects all email addresses — it is a production-wide failure, not per-address

Additionally, the error copy is contradictory and misleading:

| What the copy says | What it implies | What actually happened |
|--------------------|-----------------|------------------------|
| "Could not send the sign-in link." | There was an error | Correct — no email sent |
| "Check your email and try again." | An email was sent | Wrong — no email was sent |

Users waste time checking their inbox for an email that will never arrive, and have no actionable path forward.

---

## Root cause (confirmed)

**Supabase built-in email rate limit on `powri-prod`:**
The `powri-prod` Supabase project (Story 9.7) uses the built-in email provider, limited to **2 emails/hour for the entire project**. Setup testing and QA exhausted the cap; all `signInWithOtp` calls fail with rate-limit errors (observed as `400` + `AUTH_ERROR` on production before deploy; Supabase side: `over_email_send_rate_limit`).

Custom SMTP was considered but **deferred** — no owned domain on Vercel deployment; Resend not pursued for this story.

---

## Fix steps

### 1. Diagnose on Supabase Dashboard (`powri-prod`) — deferred

Not required to close 9.9 (error-copy scope). Retain for a future delivery/SMTP story.

### 2. Resolve email delivery — deferred

Custom SMTP / Resend **not proceeding** per PO scope decision. Track as future epic/story when Powri has an owned domain.

### 3. Fix the error copy ✅ (in scope — done)

The error message "Could not send the sign-in link. Check your email and try again." split into two distinct copy strings based on failure type.

**Case 1 — Server/delivery failure** (Supabase returned an error; no email sent):
> "Something went wrong — we couldn't send the sign-in link. Please try again in a few minutes."

**Case 2 — IP / request rate limit** (`over_request_rate_limit`; too many auth requests from this client):
> "Too many sign-in attempts. Please wait a few minutes and try again."

**Note (post-review):** `over_email_send_rate_limit` maps to Case 1, not Case 2 — Supabase uses that code for project-wide SMTP caps where no email was sent.

Mapped in `SignInSheet.tsx` + `magicLinkErrors.ts`. Updated `messages/en.json`. Raw `AUTH_ERROR` never surfaced in UI.

### 4. Re-test on production — deferred

Production delivery retest **not proceeding** per PO scope decision. Error-copy behavior verifiable after merge/deploy via Network tab (`MAGIC_LINK_RATE_LIMITED` / `MAGIC_LINK_SEND_FAILED` vs legacy `AUTH_ERROR`).

---

## Acceptance criteria

| # | Criterion | Status |
|---|-----------|--------|
| 1 | Magic link email delivered to new address on `powri.vercel.app` | **Deferred** — SMTP/delivery out of scope |
| 2 | Magic link email delivered to existing account on `powri.vercel.app` | **Deferred** — SMTP/delivery out of scope |
| 3 | Delivery failure copy does **not** say "Check your email" | **Done** |
| 4 | IP rate-limit copy does **not** imply an email was sent | **Done** |
| 5 | Raw `AUTH_ERROR` never visible in UI | **Done** |
| 6 | Bug 9.8 re-verified after this fix | **Deferred** — requires working magic-link delivery |

---

## Files involved

| File | Role |
|------|------|
| `web/src/app/api/auth/magic-link/route.ts` | Calls `signInWithOtp`; returns mapped error response to client |
| `web/src/components/auth/SignInSheet.tsx` | Renders magic link error copy to user |
| `web/messages/en.json` | i18n copy — auth error strings |
| `web/src/lib/auth/magicLinkErrors.ts` | Maps Supabase codes → API + client message keys |
| `docs/ops/supabase-prod-email-smtp.md` | Reference only — future ops; not executed for 9.9 |

---

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

**In scope — done:**
- Added `magicLinkErrors.ts` — maps `over_request_rate_limit` → `MAGIC_LINK_RATE_LIMITED` (429); `over_email_send_rate_limit` and other Supabase errors → `MAGIC_LINK_SEND_FAILED` (503). Replaces generic `AUTH_ERROR`.
- Post-review: `over_email_send_rate_limit` moved to delivery-failure path so project SMTP cap on prod does not show misleading inbox copy.
- `SignInSheet.tsx` parses API error code and shows `magicLinkRateLimited` or `magicLinkSendFailed` i18n strings.
- Unit tests in `magicLinkErrors.test.ts`.
- Lint, `test:unit`, `build` pass.

**Out of scope — PO decision (2026-07-03):**
- No custom SMTP on `powri-prod`; no Resend setup (no owned domain; Vercel-only hosting).
- No production magic-link delivery retest; AC 1, 2, 6 deferred to a future domain/SMTP story.
- [`docs/ops/supabase-prod-email-smtp.md`](../../docs/ops/supabase-prod-email-smtp.md) kept as reference only.

**Context:** Story 9.7 fixed localhost redirect only; completing 9.7 exposed bugs 9.8–9.13. Production `400` from `/api/auth/magic-link` confirms Supabase-side failure (rate limit on built-in email). Error-copy fix ships without unblocking delivery.

### File List

- `web/src/lib/auth/magicLinkErrors.ts` (new)
- `web/src/lib/auth/magicLinkErrors.test.ts` (new)
- `web/src/app/api/auth/magic-link/route.ts` (modified)
- `web/src/components/auth/SignInSheet.tsx` (modified)
- `web/messages/en.json` (modified)
- `docs/ops/supabase-prod-email-smtp.md` (new — reference only)
- `docs/ops/supabase-provisioning.md` (modified)
- `docs/qa/phase2/deploy-environment.md` (modified)

### Change Log

- 2026-07-03: Split magic-link error copy; map Supabase rate-limit vs delivery errors; add production SMTP runbook (reference).
- 2026-07-03: PO scope cut — error-code/copy only; SMTP retest and delivery ACs deferred (no owned domain).
- 2026-07-03: Code review fix — `over_email_send_rate_limit` → delivery-failure copy; neutral wait copy for IP rate limit.
