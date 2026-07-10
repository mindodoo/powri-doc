# Epic 9: Supabase Foundation & Auth

Phase 2 authentication foundation: Supabase project setup, SSR session middleware, Google OAuth and magic-link sign-in, profile/account settings, GDPR export and deletion, plus production bug fixes and prod/staging environment separation.

## Stories

- **9.1** — Supabase Project & Database Migration
- **9.2** — Supabase SSR Clients & Session Middleware
- **9.3** — Sign-In Sheet — Google OAuth & Magic Link
- **9.4** — Profile Bootstrap & Account Settings
- **9.5** — GDPR Data Export & Account Deletion
- **9.6** — Bug: Production sign-in redirects to localhost with auth_error
- **9.7** — Supabase Production / Staging Separation
- **9.8** — Bug: Magic link callback — session not established
- **9.9** — Bug: Magic link sending — AUTH_ERROR on production
- **9.10** — Bug: Google OAuth auto-selects account without picker
- **9.11** — Bug: Post-sign-in — no visual confirmation; redirect issues
- **9.12** — Bug: OAuth bad_oauth_state redirects to literal `/**` → 404
- **9.13** — Bug: Profile picture upload returns 500 on production
- **9.14** — Desktop Sign-In Modal — No-Scroll UX Fix
