# TSC Calc — Technical Revitalization Log (2026-11-21)

Full technical review + repair pass after a ~7-month gap (last real work: 2026-03-28 session; last commit: 2026-04-11 "erm"). All changes on branch `claude/obsidian-corvus-tech-review-uuld30`. This log is the handoff record for Ledger's side of the revitalization plan.

---

## Where the Project Stood Before This Pass

- **v3.1 Filter accordion: actually shipped.** CLAUDE.md still listed it "in progress," but the commits from 2026-03-28 show the HIG accordion UI, dark-mode override block, and card-header contrast fix all landed. Status table corrected.
- **Next planned feature (NOT started, awaiting owner go-ahead):** Builder food-section redesign — spec approved at `docs/superpowers/specs/2026-03-28-food-section-redesign.md`, implementation plan at `docs/superpowers/plans/2026-03-28-v3-1-filter-accordion.md` sibling. Category pills + full-width item rows, purple→teal selected state.
- **Custom domain added** (2026-04-04): `CNAME` → `calc.obsidiancorvus.com`. Could not verify DNS/liveness from the review sandbox (outbound network restricted) — **owner should confirm the domain resolves and HTTPS cert is issued in GitHub Pages settings.**
- The 2026-04-11 "erm" commit dumped local session artifacts into the public repo (server logs, PID files, brainstorm scratch HTML, and a `settings.local.json` exposing a local Windows user path). Cleaned up this pass.

## Bugs Found & Fixed

| # | Severity | What | Fix |
|---|----------|------|-----|
| 1 | **Critical** | `sw.js` served everything cache-first under a permanent cache name (`tsc-calc-v1`). Installed PWA users could be **stuck on an old version forever** — deploys never reached them. | Rewrote service worker: versioned cache (`tsc-calc-v3.1.1`), network-first for page navigations with offline fallback, cache-first + background refresh for static/CDN assets (fonts, html2canvas, icons now work offline after first use). **Process rule going forward: bump `VERSION` in `sw.js` on every deploy.** |
| 2 | High | Food **search view never showed selected state** — the check read `.catIdx` off a `Set` (always undefined), so a selected item looked unselected in search results. | Now checks `selectedFood.has(key)` correctly. |
| 3 | High | **Favorites ignored smoothie size.** A 32oz order saved to Favorites reloaded with 24oz math — silently wrong numbers, which is poison for a nutrition product. | Favorites now store `size`; loading restores it (defaults to 24oz for favorites saved before this fix). |
| 4 | Medium | Favorite names from the rename prompt were injected into `innerHTML` unescaped (self-XSS / layout breakage via `<`, `&`). | Added `escHTML()`; name + subtitle now escaped. Verified with a live `<img onerror>` payload — renders as text. |
| 5 | Medium | Share card: if `html2canvas` (CDN) hadn't loaded — e.g. first visit offline — tapping the button threw silently. | Guard added; shows a "needs internet" toast instead. |
| 6 | Low | Per-100-cal toggle was visible (but inert) for Lowest/Highest Calories goals. | Visibility now driven by the single `PER100_GOALS` constant also used by the ranking logic — toggle only appears where it changes results. |
| 7 | Low | About tab still said **Version 3.0** (header badge said v3.1). | Now 3.1 — v3.1 merge checklist item #6 fully closed. |
| 8 | Hygiene | Session litter committed to public repo (`.superpowers/` server logs/PIDs, `.claude/worktrees/` incl. local settings leaking a Windows user path). No `.gitignore` existed. | Removed from tracking; added `.gitignore` covering `.superpowers/`, `.claude/worktrees/`, `settings.local.json`, logs/PIDs, OS noise. |
| 9 | Minor | No meta description (SEO/share previews). | Added. |
| 10 | Cleanup | Dead code: legacy `onFilterGoalChange()` stub, unused `GOAL_GROUP`/`SIMPLE_GOALS` constants, duplicated comment banner. | Removed. |

## Verification

Playwright smoke test (13 assertions, all passing): food search selected-state, calculate math (Blueberry Bliss 32oz + Pea Protein + Acai Bowl = 1143 cal ✓), favorite save/escape/size-restore round-trip, per-100 toggle show/hide, accordion collapse + badge, filter top-10 render, compare render, dark mode toggle, legal accordion, version row, service worker activation. No JS errors.

## ⚠️ Open Issue — DATA INTEGRITY (needs owner + Ledger, not code)

**Did NOT touch data arrays (per project rules), but the `fat` column in `smoothies[]`, `supplements[]`, and `freshAddins[]` appears to hold *calories-from-fat*, not grams, for many items:**

- Chia Banana Boost: `fat:234` on a 680-cal smoothie — 234g of fat would be ~2,100 calories by itself. As *calories from fat*, 234 → 26g, which is plausible.
- Smoking gun: the same peanut butter appears twice — freshAddins "Peanut Butter" `cal:180, fat:126` vs foodCategories "Peanut Butter Drizzle" `cal:180, fat:14`. 126 fat-calories = 14g. Identical item, one column in calories, one in grams.
- Same pattern: Avocolada `fat:72`, Peanut Butter Cup `fat:171`, Whey Protein `fat:13.5`, Chia Seeds `fat:18`, Raw Almonds `fat:27`.
- `foodCategories[]` values look like true grams throughout.
- Several smoothie **protein** values also look high for fruit smoothies (e.g. Mocha Madness `protein:105`) and are worth a spot-check in the same pass.

**Blast radius if confirmed:** FDA label Total Fat + %DV, macro donut (fat×9 inflates fat share), Lowest Fat / Fat-per-cal filter rankings, Clean Meal Score, Compare tab — i.e., most of the product's credibility surface.

**Action:** verify against the TSC Nutrition Guide (11/06/25) held on the Ledger side; if confirmed, divide affected smoothie/supplement/add-in fat values by 9 (and re-check protein) in a dedicated, reviewed data-correction commit, then bump sw.js VERSION.

## Recommendations (owner's eyes — no action taken)

1. **Resolve the fat-data question before any marketing/B2B pitch** — the label is the product.
2. Continue v3.1 with the approved food-section redesign (spec is ready; it was next in line).
3. Keyboard accessibility: tabs/rows/toggles are `div onclick` — no keyboard or screen-reader path. Worth a pass before B2B demos (restaurant chains will have accessibility checklists).
4. The 32oz math is a linear 32/24 scale of 24oz values — fine as a v3 approximation, but sits in tension with the "never estimate" rule; consider a footnote in-app, or real 32oz data if the guide has it.
5. Consider a tiny deploy checklist in CLAUDE.md (bump badge, bump About row, bump sw VERSION) so the three version markers can't drift again.
6. Manifest `purpose: "any maskable"` on one icon entry is tolerated but non-ideal; separate `any` and `maskable` entries would render better on Android.
7. Backend proxy (Railway) from the 2026-03-14 plan remains unbuilt; Privacy Policy already references "AI analysis feature" routed through a proxy — either build it or trim that line so the policy matches reality.

## Commits This Pass

1. `chore: remove committed session artifacts, add .gitignore`
2. `fix: SW update strategy, favorites size, search selection, escaping, share guard, v3.1 polish`
3. `docs: correct v3.1 status, flag fat-column data question, add revitalization log + Ledger prompt`
