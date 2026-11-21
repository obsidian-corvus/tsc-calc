# TSC Nutrition Calculator — Claude Code Context

## Project Overview
Restaurant Nutrition Calculator PWA built under **Obsidian Corvus Strategy LLC** (Florida LLC).
- Live: https://obsidian-corvus.github.io/tsc-calc/ (custom domain via CNAME: https://calc.obsidiancorvus.com)
- GitHub: https://github.com/obsidian-corvus/tsc-calc
- IP owned by OCS LLC; provisional patent grace period starts March 7, 2026

## Tech Stack
- **Frontend**: Vanilla HTML/CSS/JS — single file (`index.html`, ~2,000 lines). No framework intentional.
- **PWA**: Offline-first via Service Worker (`sw.js`), installable
- **Fonts**: Bebas Neue + DM Sans (Google Fonts CDN)
- **Share card**: html2canvas v1.4.1 (CDN)
- **Backend (planned)**: Node.js proxy on Railway — keeps Anthropic API key off client

## Current Data
All TSC nutrition data hardcoded in `index.html` as JS arrays: `smoothies[]`, `foodCategories[]`, `supplements[]`, `freshAddins[]`.
Each item: `name, cal, fat, satFat, trans, chol, sodium, carb, fiber, sugar, protein`
Source: TSC Nutrition Guide dated 11/06/25.

⚠️ **Open data question (pending verification against source guide — do NOT change values until resolved):** several `fat` values in `smoothies[]`, `supplements[]`, and `freshAddins[]` appear to be *calories from fat* rather than grams (e.g. Chia Banana Boost `fat:234` on 680 cal; freshAddins Peanut Butter `fat:126` vs the identical food item Peanut Butter Drizzle `fat:14`). `foodCategories[]` fat values look like true grams. See `docs/revitalization-log-2026-11-21.md`.

## Feature Status (v3.0 — complete)
Builder, Smoothie Size Toggle, FDA Label, Macro Donut Chart, Favorites/Saved Orders, Goal Mode, Smart Filter (13 goals + Clean Meal Score), Per-100-Cal Toggle, Context Flags, Compare Mode, Dark Mode, Share Card (html2canvas), Legal Pages (Privacy Policy, ToS, Disclaimer in About tab)

---

## Active Build — v3.1: Apple HIG UI Rebuild

### Design Direction: Apple HIG Fusion
Keep the app's current personality (teal + orange color system, dark header, Bebas Neue font) but restructure with HIG patterns. The aesthetic is CSS, not a framework. Maintain single-file constraint.

### Core HIG Patterns to Apply
- **Grouped list rows**: white card, 0.5px dividers — replaces colored card headers for content
- **Circular checkmarks**: teal fill when selected — replaces bordered grid cards for add-ins
- **Segmented control**: for size toggle (24oz / 32oz)
- **iOS-style toggle switches**: for boolean controls (per-100-cal, etc.)
- **Section labels**: uppercase, spaced, small — replaces colored card headers
- **Warm cream background**: retained

### CSS Variables (do not change)
```css
--teal: #00897B;
--teal-dark: #00695C;
--orange: #E65100;
--orange-light: #FFF3EB;
--cream: #FDF6EC;
--dark: #1A1A1A;
--mid: #666;
--light: #E8E0D5;
--white: #FFF;
--red: #C0392B;
--green: #2E7D32;
```

### Per-Tab UI Priority

| Tab | Priority | Status |
|-----|----------|--------|
| Filter — accordion goal selector | **High** | ✅ Shipped (v3.1) |
| Builder — grouped list add-ins / food redesign | High | ⏳ Pending — spec at `docs/superpowers/specs/2026-03-28-food-section-redesign.md` |
| Label — macro donut + dark header | Medium | ⏳ Pending |
| Saved — grouped list rows | Medium | ⏳ Pending |
| Goals — grouped input rows | Low | ⏳ Pending |
| Compare — two grouped cards | Low | ⏳ Pending |
| About — disclosure arrow legal rows | Low | ⏳ Pending |

---

## Filter Tab — HIG Accordion Spec (CURRENT TASK)

### What to Replace
Remove the `<select>` dropdown (`#filterGoalSelect`) and the `.filter-goal-desc` paragraph. Replace with accordion category groups.

### New Structure
Three accordion groups inside the Filter card body, above the menu type toggle:

**Group 1: Simple Goals** (7 goals)
- Most Protein
- Lowest Calories
- Highest Calories
- Most Fiber
- Lowest Sugar
- Lowest Sodium
- Lowest Fat

**Group 2: Efficiency Ratios** (5 goals)
- Best Protein/Cal Ratio
- Best Protein/Carb Ratio
- Best Fiber/Cal Ratio
- Lowest Sugar/Cal Ratio
- Lowest Fat/Cal Ratio

**Group 3: Composite** (1 goal)
- Best "Clean Meal" Score

### Accordion Behavior
- All groups start **expanded** on load
- Tapping a group header collapses/expands it
- Only one group can be open at a time (opening one closes others)
- Each group header shows: group name (left) + chevron (right, rotates on open/close)
- When a group is **collapsed with an active selection**: show a teal pill badge in the header displaying the active goal name

### Goal Row Structure
Each goal is a tappable row:
```
○  Goal Name         short description text
```
- Left: circular radio button — empty border when unselected, teal fill + white checkmark when selected
- Goal name bold (13px) + description muted (11px) on same line or below
- Full row is tappable
- Selected row gets subtle teal-tinted background (`#e8f5f3` light / `#0a2e28` dark)

### Per-100-Cal Toggle
- iOS-style toggle switch (same `.toggle-switch` pattern already in codebase)
- Positioned below accordion groups, above menu type toggle
- Only active for Simple Goals — hide when a Ratio or Composite goal is selected
- Label: "Per 100 cal"

### Menu Type Toggle (Smoothies / Food / Both)
- Stays as segmented control, no logic changes
- Moves below per-100-cal toggle

### JS Changes
- Replace `filterGoal = document.getElementById('filterGoalSelect').value` with row-tap setter
- `onFilterGoalChange()` still fires, triggered by row tap instead of select change
- `runFilter()` logic completely unchanged
- Per-100-cal toggle: show/hide based on whether `filterGoal` is in `simplePer100Goals`

### Dark Mode for New Filter UI
- Accordion group headers: `background: #2a2a2a`, border `#3a3a3a`
- Group header text: `color: #f0f0f0`
- Goal rows: `background: #2a2a2a`, dividers `#3a3a3a`
- Selected goal row: `background: #0a2e28`
- Circular radio unselected: border `#3a3a3a`; selected: `var(--teal)` fill
- Teal pill badge: same teal, readable both modes
- Description text: `#aaa` in dark mode

---

## Dark Mode Rules (Critical — Do Not Break)

### The Root Cause — Variable Reassignment Trap
```css
body.dark-mode {
  --white: #2a2a2a;  /* surfaces become dark */
  --cream: #1e1e1e;  /* backgrounds become darker */
  --light: #3a3a3a;  /* borders become dark */
  --mid:   #aaa;     /* muted text becomes lighter */
}
```

### Golden Rules
1. **NEVER use `var(--white)` for text on colored backgrounds** — becomes `#2a2a2a` in dark mode
2. Always hardcode `#ffffff` for text that must stay white (card header titles)
3. Every white/cream surface needs a `body.dark-mode` background override → `#2a2a2a`
4. Every light border needs a dark override → `#3a3a3a`
5. Text using `var(--dark)` needs explicit override → `color: #f0f0f0`
6. FDA label stays white in dark mode — intentional, never override
7. Context flag tags need both background AND color overrides

### Key Color Reference

| Element | Light | Dark |
|---------|-------|------|
| Card background | `#fff` | `#2a2a2a` |
| Page background | `#fdf6ec` | `#1a1a1a` |
| Borders | `#e8e0d5` | `#3a3a3a` |
| Body text | `#1a1a1a` | `#f0f0f0` |
| Muted text | `#666` | `#aaa` |
| Header text | `#ffffff` hardcoded | `#ffffff` hardcoded |
| Teal / Orange | unchanged | unchanged |

### Checklist for Every New Component
- [ ] White/cream surfaces have `body.dark-mode` background override
- [ ] Text using `var(--dark)` has `color: #f0f0f0` override
- [ ] Text on colored background uses `#ffffff` hardcoded
- [ ] Border/divider colors have `#3a3a3a` override
- [ ] Pastel tags/badges have both background AND color dark overrides

---

## Merge Requirements for Current Build (v3.1)

The `index.html` in the repo is v3.0. The new build must:

1. **Carry forward all dark mode fixes** from the POR — the full expanded `body.dark-mode` override block covering: legal panels, about tab, compare tab, food categories, section labels, buttons, goal mode, favorites, filter, size toggle, smoothie preview, tags, alerts, toast, field labels, macro chart legend
2. **Carry forward the legal panels** from the current repo's About tab — Privacy Policy, ToS, Disclaimer accordion panels with `toggleLegal()` function and all associated CSS
3. **Replace the Filter tab goal selector** with the new HIG accordion UI per spec above
4. **Do not remove or break any existing features**
5. **Do not change the data arrays** — smoothies, supplements, freshAddins, foodCategories are correct and final
6. **Bump version badge** from `v3.0` to `v3.1`

---

## Architecture Decisions
- Vanilla JS over React — single-file, offline-first, zero build step
- Railway over Render — no cold starts on free tier ($5/mo credit)
- PDF upload over DB integration — no partnerships, works with public data
- B2B primary monetization — direct restaurant chain pitches
- FDA label stays white in dark mode — legibility priority
- Per-100-cal toggle: simple goals only (ratio goals already normalize by cal)
- NEVER estimate nutritional data — show "N/A" if unavailable
- Single-file constraint is intentional — preserve it unless multi-restaurant rebuild starts

## Legal Pages (Completed — in About Tab)
- Privacy Policy, ToS, Disclaimer as accordion panels inside About tab
- `toggleLegal(id)` function handles accordion behavior
- All panels use existing CSS variables — dark mode automatic

## USDA FoodData Central (Future)
- API: https://fdc.nal.usda.gov/food-search
- Free, no auth for basic queries
- Use for micronutrient gaps only — always label as "USDA estimate"
- Never interpolate; show "N/A" when data unavailable

## Code Style
- Inline styles in `<style>` block at top of index.html
- CSS variables: `--teal`, `--light`, `--mid`, etc.
- Tab switching via `switchTab(name)` function
- No build tools, no transpilation — pure browser JS
- Emoji icons used throughout UI intentionally
