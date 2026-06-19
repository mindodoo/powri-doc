# PRD: Quiz (FR-4) & Onboarding (FR-5)

**Source:** [`../prd.md`](../prd.md) FR-4, FR-5, §12.5 Screen 0/0b  
**Feature brief:** [`../features/quiz.md`](../features/quiz.md) ← **start here for quiz implementation**  
**Epic:** [`../epics/epic-02-discover.md`](../epics/epic-02-discover.md) Stories 2.1, 2.7

---

## FR-4 — Find My Resort Quiz

**Maps to:** Screen 0b (§12.5)

### Acceptance criteria
- [ ] 4-question sliding-card quiz from onboarding and Home/Explore
- [ ] **Serious / Cosmic mode toggle** before Q1 (default Serious); both map to same scoring enums
- [ ] Ranking per [`docs/quiz-scoring.md`](../../../docs/quiz-scoring.md)
- [ ] **Results reveal:** Serious header only; Cosmic interstitial + header + subline (required — `features/quiz.md` § Results reveal)
- [ ] **Results podium:** top 3 only — featured #1 + minimal #2/#3; match reason on #1; Browse CTA → Explore (`features/quiz.md` § Results podium)
- [ ] `quiz_started`, `quiz_completed` per tracking plan (`quiz_completed` after Cosmic reveal completes; **`result_count: 3`**)

### Screen 0b essentials

- Sliding card quiz — **not** a form. One question per full-screen card; swipe/tap → next slides from right
- **4 questions max** per mode — full copy in [`../features/quiz.md`](../features/quiz.md)
- **Serious mode:** direct ski-trip questions (PRD canonical table)
- **Cosmic mode:** playful oblique questions; 1:1 enum mapping to Serious (see feature brief)
- **Single scoring path:** `score.ts` reads `level`, `priority`, `region`, `timing` enums only
- Results headers differ by mode; **podium layout identical** for equivalent answers (top 3 only on quiz screen)
- **Podium:** #1 featured with match reason; #2/#3 minimal rows; *"Browse all resorts →"* → Explore
- `score.ts` ranks all 20; ranks 4–20 not shown on quiz results
- Answer buttons: **full-width**, easy mobile tap
- Phase 3: save quiz preferences to user account (deferred)

---

## FR-5 — Onboarding

**Maps to:** Screen 0 (§12.5)

### Acceptance criteria
- [ ] ≤4 swipeable full-screen cards; skip + "Get started"
- [ ] Not shown again after completion (local flag)
- [ ] No hero photography (fast load)

### Screen 0 essentials

- 3–4 cards: app intro, find resort, plan trip, honest resort guides *(no AI card in Phase 1)*
- Progress dots; skip top-right; "Get started" on final card
- Final card offers quiz entry (links to Explore/quiz flow)
- Background: `--color-bg` warm off-white; icons only — no photos
- Tone: warm, welcoming — not "Sign up to access features"

**Full screen spec:** [`13-ux-screens.md`](13-ux-screens.md#screen-0-onboarding)

---

## Components

| Component | UX-DR | Notes |
|-----------|-------|-------|
| `QuizModeToggle` | — | Serious / Cosmic before Q1 |
| `QuizResultsReveal` | — | Mode-specific header; Cosmic interstitial |
| `QuizTopMatchCard` | — | Featured #1 — accent border, dual labels, match reason |
| `QuizRunnerUpRow` | — | Minimal #2/#3 — 48px thumb, name, region |
| `OnboardingCard` | UX-DR11 | Full-screen swipeable |
| `ResortQuizCard` | UX-DR12 | Wide answer buttons, 52px min height |

**Design tokens:** [`../ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md`](../ux-designs/ux-japan_winter_sport-2026-06-12/DESIGN.md)
