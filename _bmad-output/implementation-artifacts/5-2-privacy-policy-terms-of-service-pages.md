# Story 5.2: Privacy Policy & Terms of Service Pages

Status: done

baseline_commit: 48d7544

## Story

As a **user and legal stakeholder**,
I want accessible Privacy Policy and Terms pages linked from Info,
So that the product meets GDPR/PDPA before public sharing.

## Acceptance Criteria

1. **Given** Info tab legal section  
   **When** I tap Privacy Policy or Terms of Service  
   **Then** dedicated pages render at `/privacy` and `/terms` (FR-7, §17.1)

2. **Given** the Privacy Policy page  
   **When** rendered  
   **Then** it covers analytics (§13), third parties (Anthropic, PostHog, Plausible, Unsplash), consent, retention, and user rights

3. **Given** the Terms page  
   **When** rendered  
   **Then** it includes "information as-is, verify against official resort sources" disclaimer (§17.1, R5)

4. **Given** legal pages  
   **When** viewed on mobile  
   **Then** they are readable without external dependencies (no iframes/embeds)

## Tasks / Subtasks

- [x] **Legal layout component** (AC: 4)
  - [x] `LegalPageLayout` — back to Info, title, last updated, section typography

- [x] **Privacy + Terms pages** (AC: 2, 3, 4)
  - [x] `/privacy` and `/terms` routes under `[locale]`
  - [x] Section content in i18n (`legal.privacy`, `legal.terms`)

- [x] **Info tab links** (AC: 1)
  - [x] Legal section with links to `/privacy` and `/terms`

- [x] **Validate**
  - [x] `npm run build` + `npm run lint`

## Dev Notes

- **Contact email:** inject `getContactEmail()` into privacy rights section via i18n `{email}` param
- **Anthropic:** disclose for future AI (Phase 3); Phase 1 has no live AI chat UI
- **Supabase/Google:** not listed in Story 5.2 AC — omit or note "not used Phase 1"
- **Out of scope:** satisfaction survey (5.3), analytics verification (5.4)

### References

- [Source: `_bmad-output/planning-artifacts/epics.md` Story 5.2]
- [Source: `_bmad-output/planning-artifacts/prd/15-analytics-legal-glossary.md` §17]
- [Source: `docs/tracking-plan.md` — analytics principles]

## Dev Agent Record

### Agent Model Used

Composer

### Completion Notes

- `/privacy` and `/terms` pages with mobile-readable section layout (no embeds)
- Privacy covers analytics, consent, PostHog/Plausible/Unsplash/Anthropic, retention, GDPR/PDPA rights
- Terms includes as-is disclaimer, verify official sources, liability, acceptable use, governing law
- Info tab Legal section links to both pages

### File List

- `web/messages/en.json` (modified — `info.legal*`, `legal.*`)
- `web/src/lib/legal/sections.ts` (new)
- `web/src/components/legal/LegalPageLayout.tsx` (new)
- `web/src/app/[locale]/privacy/page.tsx` (new)
- `web/src/app/[locale]/terms/page.tsx` (new)
- `web/src/app/[locale]/info/page.tsx` (modified — legal links)

## Senior Developer Review (AI)

**Review date:** 2026-06-13  
**Outcome:** Approve

### Summary

All four ACs satisfied. Info links to `/privacy` and `/terms`; both pages render self-contained legal copy covering required topics; mobile typography matches app tokens; no external dependencies.

### Review Findings

- [x] [Review][Pass] Privacy: analytics, third parties (Anthropic/PostHog/Plausible/Unsplash), consent, retention, rights with `{email}` injection
- [x] [Review][Pass] Terms: as-is disclaimer + verify official sources (R5)
- [x] [Review][Pass] Legal links use `min-h-11` tap targets; back link returns to `/info`
- [x] [Review][Defer] Legal review by qualified counsel before Ring 2 public share — content is Phase 1 scaffold
- [x] [Review][Defer] Update `LEGAL_LAST_UPDATED` when processors or features change
- [x] [Review][Note] Anthropic disclosed for Phase 3 AI; accurate for Phase 1 (no live chat UI)
