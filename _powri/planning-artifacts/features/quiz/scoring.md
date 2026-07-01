# Quiz Scoring — Find My Resort

**Status:** Draft (Story 2.7)  
**Owner:** Product / Engineering  
**Companion to:** [`_powri/planning-artifacts/features/quiz/brief.md`](brief.md), [`docs/analytics/tracking-plan.md`](../../../../docs/analytics/tracking-plan.md), [`content/content-model.md`](../../../../content/content-model.md)

---

## 0. How to use this file

This is the **single source of truth** for quiz ranking logic. The feature brief defines question copy and enums; **this file defines how enums become a sorted resort list**.

**Rules:**

1. **Serious and Cosmic modes share this algorithm.** Only display strings differ; `score.ts` never branches on `quiz_mode`.
2. **Score from content fields**, not display strings. Primary inputs: `region`, `bestFor`, `bestMonths`, `seasonTypical` on `Resort` (see `web/src/lib/content/types.ts`).
3. **Reuse tag-matching style** from `web/src/lib/resort/filters.ts` (`normalizeTag`, substring `includes` checks).
4. **Tune weights here first**, then update `score.ts` in the same change.
5. **Validate with persona fixtures** (§6) before Story 2.7 sign-off.

---

## 1. Input & output

### Input — `QuizAnswers`

Stable enums from [`features/quiz/brief.md`](brief.md). TypeScript names for implementers:

```typescript
export type QuizLevel = 'beginner' | 'confident' | 'advanced' | 'group';
export type QuizPriority =
  | 'powder_terrain'
  | 'village_culture'
  | 'family'
  | 'all';
export type QuizRegion = 'hokkaido' | 'honshu' | 'tohoku' | 'unsure';
export type QuizTiming =
  | 'early_season'
  | 'peak_february'
  | 'late_season'
  | 'still_planning';

export type QuizAnswers = {
  level: QuizLevel;
  priority: QuizPriority;
  region: QuizRegion;
  timing: QuizTiming;
};
```

### Output

```typescript
export type ScoredResort = {
  slug: string;
  score: number; // 0–100 theoretical max; typically 40–95
};

// Sorted descending by score; tie-break by slug ascending (stable, deterministic)
export function scoreResorts(
  resorts: Resort[],
  answers: QuizAnswers,
): ScoredResort[];
```

Results UI maps slugs back to `ResortListItem[]` via the existing resort index. **Show all 20 resorts ranked** (not a hard filter) unless score is 0 — see §5 fallback.

---

## 2. Scoring model

Each resort earns **0–100 points** across four axes. Sum axis scores.

| Axis | Max points | Rationale |
|------|------------|-----------|
| **Region** (Q3) | 35 | Gateway / flight corridor is the strongest practical constraint |
| **Priority** (Q2) | 30 | Trip character — powder vs culture vs family |
| **Level** (Q1) | 25 | Ability / intensity fit |
| **Timing** (Q4) | 10 | Soft signal — most resorts peak in Feb; still differentiates early/late |

```typescript
total =
  scoreRegion(resort, answers.region) +
  scorePriority(resort, answers.priority) +
  scoreLevel(resort, answers.level) +
  scoreTiming(resort, answers.timing);
```

---

## 3. Axis rules

### 3.1 Region (`max 35`)

Maps quiz gateway choice to `Resort.region` (content frontmatter).

| `QuizRegion` | Matching `Resort.region` values |
|--------------|----------------------------------|
| `hokkaido` | `Hokkaido` |
| `honshu` | `Nagano`, `Niigata` *(Tokyo / Nagano corridor — Shinkansen & Narita access)* |
| `tohoku` | `Tohoku` |
| `unsure` | No penalty — see neutral rule below |

| Condition | Points |
|-----------|--------|
| Region matches selected gateway | **35** |
| Region does not match | **0** |
| `unsure` selected | **17.5** for every resort *(neutral — other axes decide)* |

**Resort inventory by region (Phase 1):**

| Region | Slugs |
|--------|-------|
| Hokkaido | `niseko-united`, `furano`, `rusutsu`, `kiroro`, `sahoro`, `tomamu` |
| Nagano | `hakuba-valley`, `shiga-kogen`, `nozawa-onsen`, `madarao-kogen`, `tsugaike-kogen` |
| Niigata | `myoko-kogen`, `naeba`, `gala-yuzawa`, `joetsu-kokusai`, `lotte-arai`, `kandatsu-kogen` |
| Tohoku | `zao-onsen`, `appi-kogen`, `nekoma-alts-bandai` |

---

### 3.2 Priority (`max 30`)

Uses `resort.bestFor` tags (case-insensitive substring match, same pattern as `filters.ts`).

Helper: `tagsInclude(tags, ...needles)` → true if any tag contains any needle.

#### `powder_terrain`

| Condition | Points |
|-----------|--------|
| Tag matches `powder` | **20** |
| Tag matches any of `tree runs`, `backcountry`, `freeride`, `big terrain`, `steep`, `snow quality` | **+10** |
| Cap | **30** |

#### `village_culture`

| Condition | Points |
|-----------|--------|
| Tag matches any of `village charm`, `onsen culture`, `traditional`, `scenery` | **20** |
| Tag matches `onsen` *(in tag text)* | **+5** |
| Tag matches `nightlife` or `international` | **+5** |
| Cap | **30** |

#### `family`

| Condition | Points |
|-----------|--------|
| Tag matches `families` or `family` | **20** |
| Tag matches any of `beginners`, `kids`, `all-inclusive`, `convenience` | **+10** |
| Cap | **30** |

#### `all` *(balanced — reward breadth)*

Count how many of these three buckets hit (each bucket at most once):

| Bucket | Match if tag includes |
|--------|----------------------|
| Powder / terrain | `powder`, `tree runs`, `big terrain` |
| Culture / village | `village`, `onsen`, `traditional`, `scenery` |
| Family / ease | `families`, `family`, `beginners`, `all levels`, `convenience` |

| Buckets hit | Points |
|-------------|--------|
| 0 | **5** |
| 1 | **15** |
| 2 | **24** |
| 3 | **30** |

---

### 3.3 Level (`max 25`)

Uses `resort.bestFor` tags.

#### `beginner`

| Condition | Points |
|-----------|--------|
| Tag matches `beginners` or `beginner` | **15** |
| Tag matches `all levels` | **+10** |
| Tag matches `families` or `family` | **+5** |
| Tag matches `groomers` | **+5** |
| Cap | **25** |

#### `confident`

| Condition | Points |
|-----------|--------|
| Tag matches `intermediates` or `intermediate` | **15** |
| Tag matches `all levels` | **+10** |
| Tag matches `groomers` or `long runs` | **+5** |
| Cap | **25** |

#### `advanced`

| Condition | Points |
|-----------|--------|
| Tag matches `powder` | **10** |
| Tag matches any of `tree runs`, `backcountry`, `freeride`, `steep`, `advanced` | **+15** |
| Tag matches `fewer crowds` | **+5** |
| Cap | **25** |

#### `group` *(mixed ability)*

| Condition | Points |
|-----------|--------|
| Tag matches `all levels` | **15** |
| Tag matches `families` or `family` | **+10** |
| Tag matches any of `big terrain`, `ski-in ski-out`, `convenience` | **+5** |
| Cap | **25** |

---

### 3.4 Timing (`max 10`)

Uses `resort.bestMonths` and `resort.seasonTypical` strings (case-insensitive).

#### `early_season` (Dec–Jan)

| Condition | Points |
|-----------|--------|
| `seasonTypical` includes `december` or `dec` | **10** |
| `seasonTypical` includes `november` or `nov` | **+3** *(early open)* |
| Cap | **10** |

#### `peak_february`

| Condition | Points |
|-----------|--------|
| `bestMonths` includes `february` or `feb` | **10** |
| `bestMonths` includes `january` or `jan` | **+3** *(shoulder to peak)* |
| Cap | **10** |

#### `late_season` (Mar–Apr)

| Condition | Points |
|-----------|--------|
| `bestMonths` includes `march` or `mar` | **8** |
| `seasonTypical` includes `april`, `apr`, or `may` | **+4** |
| `bestMonths` includes `january–march` or `january–february–march` pattern | **+2** |
| Cap | **10** |

#### `still_planning`

| Condition | Points |
|-----------|--------|
| Any timing | **5** *(flat — defer to other axes)* |

---

## 4. Results presentation

1. Sort all published resorts by `score` descending (full ranking — all 20 computed).
2. **Exclude** resorts with `score === 0` only if every resort would still leave ≥3 results; otherwise include them at the bottom.
3. **Display top 3 only** on the quiz results screen — podium layout (§4.2). Ranks 4–20 are **not shown** on quiz results; Home/Explore/search unchanged.
4. **Results reveal** by mode (required — Story 2.7):
   - **Serious:** header only → podium (`quiz.results.serious`)
   - **Cosmic:** ~800ms interstitial (`quiz.reveal.cosmic.loading`) → header (`quiz.results.cosmic`) + subline (`quiz.results.cosmicSubline`) → podium
5. **Browse CTA** below podium: *"Browse all resorts →"* (`quiz.results.browseAll`) → Explore tab.
6. Fire `quiz_completed` **after** results header is visible (Cosmic: after interstitial completes).

`result_count` on `quiz_completed` = **3** (resorts displayed on podium).

### 4.1 Match reason (#1 only)

`getMatchReason(resort, axisScores, userAnswers)` in `web/src/lib/quiz/matchReason.ts`:

1. For the #1 resort, use per-axis points already computed during scoring (Region 0–35, Priority 0–30, Level 0–25, Timing 0–10).
2. Pick the **highest-scoring axis** for that resort.
3. Tie-break order: **priority → region → level → timing** (descending weight).
4. Map axis + user enum to an i18n template under `quiz.results.matchReason.*`.

Example outputs (Serious):

| Dominant axis | User answer | Copy |
|---------------|-------------|------|
| `priority` | `powder_terrain` | Strong match — powder & terrain |
| `priority` | `village_culture` | Strong match — village & culture |
| `region` | `hokkaido` | Strong match — Hokkaido |
| `level` | `advanced` | Strong match — advanced terrain |
| `timing` | `peak_february` | Good match — peak February |

Cosmic mode uses the same logic and keys — copy stays factual (resort data), not cosmic-themed.

Display on `QuizTopMatchCard` only — accent typography, one line, below region/name.

### 4.2 Podium layout

| Rank | Component | Detail level |
|------|-----------|--------------|
| **#1** | `QuizTopMatchCard` | Featured: accent border, rank **"1"** + label (*Your top match* / *Your cosmic #1*), taller hero (`16/9`), full tagline + badges, **match reason** |
| **#2** | `QuizRunnerUpRow` | Minimal: 48×48 thumb, name, region, rank badge — no tagline/badges/match reason |
| **#3** | `QuizRunnerUpRow` | Same as #2 |

Full spec: [`features/quiz/brief.md`](brief.md) § Results podium.

**No answer summary** tags under the results header.

---

## 5. Fallback when results feel too narrow

If fewer than **3** resorts score above **40** total points:

1. Re-run ranking with **region axis set to `unsure` behaviour** (17.5 for all) while keeping other axes.
2. If still <3 above 40, show top **5** by score regardless of region match.

Log internally (dev console Phase 1) when fallback triggers — future analytics hook optional.

---

## 6. Persona validation fixtures

Run these through `scoreResorts()` before sign-off. **Pass criteria** in the right column.

### Ninja (expert · powder · Hokkaido · February)

```typescript
{
  level: 'advanced',
  priority: 'powder_terrain',
  region: 'hokkaido',
  timing: 'peak_february',
}
```

| Pass | Requirement |
|------|-------------|
| ✅ | At least one of **`rusutsu`**, **`kiroro`**, **`niseko-united`** in **top 3** |
| ✅ | All top 5 are **Hokkaido** region |
| ✅ | Ranking is **defensible** — Rusutsu/Kiroro score ≥ Niseko is acceptable (fewer crowds / tree runs) |

*Cosmic equivalent enums produce identical scores.*

### Alex (confident · family · Hokkaido · still planning)

```typescript
{
  level: 'confident',
  priority: 'family',
  region: 'hokkaido',
  timing: 'still_planning',
}
```

| Pass | Requirement |
|------|-------------|
| ✅ | Top 5 include **`furano`**, **`sahoro`**, and/or **`tomamu`** (family-forward Hokkaido) |
| ✅ | No Tohoku/Niigata resorts in top 5 |

### Culture seeker (Honshu · village)

```typescript
{
  level: 'confident',
  priority: 'village_culture',
  region: 'honshu',
  timing: 'peak_february',
}
```

| Pass | Requirement |
|------|-------------|
| ✅ | **`nozawa-onsen`** in top 5 |
| ✅ | Top 5 are Nagano and/or Niigata only |

### Mind (intermediate · powder + village/food · anti-crowd · February)

**Profile:** Intermediate snowboarder; loves Japan on and off the mountain. Values food, village vibe, and walkable bases as much as riding. Prefers ski areas integrated with the village, local high-rated food, fewer crowds, and bluebird powder days.

**Serious quiz mapping:** Getting confident · All of the above · Not sure yet · Peak powder (Feb)  
**Cosmic mapping:** still asleep · twenty minutes, one lap · already at the park · depends who's asking · greedy for everything · blank postcard · *The Week Everything Peaks*

```typescript
{
  level: 'confident',
  priority: 'all',
  region: 'unsure',
  timing: 'peak_february',
}
```

| Pass | Requirement |
|------|-------------|
| ✅ | **`nozawa-onsen`** in **top 3** |
| ✅ | **`hakuba-valley`** in **top 3** |
| ✅ | **`madarao-kogen`** in **top 5** *(powder + fewer crowds + intermediates)* |
| ✅ | Top 5 include at least **two** of: `nozawa-onsen`, `madarao-kogen`, `hakuba-valley`, `zao-onsen` |

*Optional Honshu variant:* same answers with `region: 'honshu'` — expect **`nozawa-onsen`** #1–2 and Hokkaido resorts outside top 10.

*Known gap (Phase 1):* `fewer crowds` is not weighted on `all` / `village_culture` priority — Niseko may rank above Rusutsu/Kiroro despite Mind's crowd preference. Tuning deferred post-launch.

### Unsure gateway

```typescript
{
  level: 'group',
  priority: 'all',
  region: 'unsure',
  timing: 'still_planning',
}
```

| Pass | Requirement |
|------|-------------|
| ✅ | Results span **≥3 regions** in top 10 |
| ✅ | No single region monopolises all top 3 unless scores tie within 5 points |

---

## 7. Implementation checklist (Story 2.7)

- [ ] `web/src/lib/quiz/types.ts` — enums + `QuizAnswers`
- [ ] `web/src/lib/quiz/questions.ts` — Serious/Cosmic copy → shared `enumValue` per option
- [ ] `web/src/lib/quiz/score.ts` — axis functions per §3
- [ ] `web/src/lib/quiz/score.test.ts` — Ninja, Alex, and Mind fixtures (§6)
- [ ] `useQuizStore` — `quizMode`, `answers`, `scoreResorts()` on complete
- [ ] Results page reuses `ResortList` / `ResortCard` with `list_context: quiz`
- [ ] `web/src/components/quiz/QuizResultsReveal.tsx` — Serious header; Cosmic interstitial + header + subline
- [ ] Analytics: `quiz_completed` after reveal visible (Cosmic: post-interstitial)

---

## 8. Phase 2+ (out of scope)

- Persist last quiz answers / preferred mode to user profile
- Weight tuning from PostHog (`quiz_completed` → `resort_card_tapped` conversion by rank position)
- ML re-ranking — not Phase 1

---

## 9. Change log

| Date | Change |
|------|--------|
| 2026-06-12 | Initial scoring spec — Serious/Cosmic shared enums, 4-axis weighted model, persona fixtures |
| 2026-06-12 | Added Mind persona fixture (intermediate · powder + village · anti-crowd) |
