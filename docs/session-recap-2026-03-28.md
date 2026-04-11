# TSC Calc — Session Recap (2026-03-28)

## Project Context

Restaurant Nutrition Calculator PWA — single file (`index.html`, ~2,100 lines), vanilla HTML/CSS/JS, no build step. Live at https://obsidian-corvus.github.io/tsc-calc/. Active build is **v3.1** (Apple HIG UI Rebuild).

Repo: https://github.com/obsidian-corvus/tsc-calc

---

## What Was Shipped Today (pushed to main)

### 1. Dark Mode — Full Override Block
Carried forward the complete `body.dark-mode` CSS override block from the POR (point-of-reference) file. Now covers: legal panels, about tab, compare tab, food categories, section labels, buttons, goal mode, favorites, filter, size toggle, smoothie preview, tags, alerts, toast, field labels, macro chart legend. Removed two orphaned/duplicate dark mode rules.

### 2. Filter Tab — HIG Accordion UI
Replaced the `<select>` dropdown goal picker with a three-group accordion:
- **Simple Goals** (7): Most Protein, Lowest/Highest Calories, Most Fiber, Lowest Sugar/Sodium/Fat
- **Efficiency Ratios** (5): Best Protein/Cal, Protein/Carb, Fiber/Cal, Lowest Sugar/Cal, Fat/Cal
- **Composite** (1): Best "Clean Meal" Score

Behavior: groups expand/collapse exclusively, chevron rotates, teal pill badge appears in header when group is collapsed with an active selection. Per-100-cal toggle moved into the filter card and auto-hides for ratio/composite goals (already normalised by cal). Full JS refactor to named module-scope functions: `selectGoalRow`, `toggleAccGroup`, `updateAccordionBadges`, `updatePer100Visibility`.

### 3. Dark Mode Contrast Fix
`.card-header-title` was using `var(--white)` which becomes `#2a2a2a` in dark mode — near-invisible on colored card headers. Fixed by hardcoding `#ffffff` via `body.dark-mode .card-header-title`.

### 4. Version Bump + Cleanup
Badge updated to `v3.1`, `index-por.html` deleted.

---

## In Progress — Next Session

### Food Section Redesign (Builder Tab, Step 3 — Add Food)

**Spec:** `docs/superpowers/specs/2026-03-28-food-section-redesign.md`

**What's changing:** Replace the current 2-column `.addin-card` grid and plain category headers with compact full-width rows and pill-style category headers.

**Design (approved):**

```
[ BOWLS   1 selected ]           9 items ▲
  ✓  Acai Bowl                         560
  ○  Bahama Mama Bowl                  400
  ○  Dragon Fruit Bowl                 460
[ BREAKFAST ]                    6 items ▼
[ WRAPS ]                        8 items ▼
```

- **Category pills** (`id="catpill_{ci}"`): rounded pill header, item count on right, teal "N selected" badge when items picked, exclusive open (one at a time)
- **Item rows** (`.item-row`, `id="foodrow_{ci}_{ii}"`): full-width, circular checkbox (`.item-check`), food name + calorie badge
- **Selected state**: teal (replaces current purple) — row bg `#e8f5f3` / `#0a2e28` dark, teal text, filled checkbox with `✓`
- **Multi-select**: already built — `selectedFood` is a `Set`, no logic change needed
- **Tags strip** (`#selectedFoodTags`): kept, colors updated to teal

**JS functions to update:**
- `renderFoodCategories()` — generate new pill + row markup
- `toggleFoodCat(ci)` — close others when opening one
- `selectFood(ci, ii, rowEl)` — toggle `.sel`, update checkbox, update badge count
- `reset()` — add querySelectorAll for `.item-row`, `.item-check`, `.item-radio-check`, `.cat-sel-badge`
- `loadFavorite()` — update element reference from `foodcard_{ci}_{ii}` to `foodrow_{ci}_{ii}`

**Important:** Search view (`searchFood()`) still uses `.addin-card` + `.food-selected` — those CSS classes must NOT be removed. Search view redesign is a separate future task.

**CSS classes to add:** `.cat-pill`, `.cat-pill.open`, `.cat-pill-left`, `.cat-name`, `.cat-sel-badge`, `.cat-right`, `.item-list`, `.item-list.open`, `.item-row`, `.item-row.sel`, `.item-check`, `.item-radio-check`, `.item-name`, `.item-cal` — plus full dark mode overrides for each.

**CSS classes to remove** (no longer used in category render path): `.food-cat-header`, `.food-cat-divider`, `.food-cat-name`, `.food-cat-arrow`, `body.dark-mode .food-cat-body`

---

## CSS Variable Reference (do not change)

```css
--teal: #00897B;
--orange: #E65100;
--cream: #FDF6EC;
--dark: #1A1A1A;
--mid: #666;
--light: #E8E0D5;
--white: #FFF;
```

Dark mode reassigns: `--white → #2a2a2a`, `--cream → #1e1e1e`, `--light → #3a3a3a`, `--mid → #aaa`. **`--dark` is NOT reassigned** — never use `var(--dark)` for text in dark mode, hardcode `#f0f0f0` instead.
