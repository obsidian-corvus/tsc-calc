# v3.1 Filter Accordion Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade index.html from v3.0 to v3.1 by expanding the dark mode block, keeping legal panels, replacing the Filter tab `<select>` goal picker with an HIG accordion UI, and bumping the version badge.

**Architecture:** All changes are in the single `index.html` file. CSS changes (dark mode expansion + new accordion styles) land in the `<style>` block. HTML changes replace only the filter card goal-selector area. JS changes update `onFilterGoalChange()`, add accordion behavior, and add per-100-cal show/hide logic. No other tabs, data arrays, or existing features are touched.

**Tech Stack:** Vanilla HTML/CSS/JS — single file, no build step.

---

## Chunk 1: Dark Mode Block Expansion

### Task 1: Replace the partial dark mode block with the full POR block

**Files:**
- Modify: `index.html:38-82` (dark mode variables + basic overrides)
- Modify: `index.html:370` (orphaned `.filter-goal-select` dark rule — remove)
- Modify: `index.html:424` (lone `.legal-panel` box-shadow dark rule — merge into expanded block)

- [ ] **Step 1: Replace lines 38–82** (current partial `body.dark-mode` block) with the full expanded block below.

  Find this exact block (lines 38–82 in index.html):
  ```css
    body.dark-mode {
      --cream: #1e1e1e;
      --white: #2a2a2a;
      --light: #3a3a3a;
      --mid: #aaa;
      --dark-text: #f0f0f0;
      --surface: #2a2a2a;
      --surface2: #1e1e1e;
      --border: #3a3a3a;
      --text: #f0f0f0;
      --text-mid: #aaa;
    }

    body.dark-mode .scroll-area { background: #1a1a1a; }
    body.dark-mode .card { background: #2a2a2a; box-shadow: 0 2px 12px rgba(0,0,0,0.3); }
    body.dark-mode select { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%23aaa' stroke-width='2' fill='none' stroke-linecap='round'/%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 12px center; }
    body.dark-mode .smoothie-preview { background: #333; }
    body.dark-mode .addin-card { background: #333; border-color: #444; }
    body.dark-mode .addin-name { color: #f0f0f0; }
    body.dark-mode .food-toggle { background: #333; }
    body.dark-mode .food-toggle-btn.active { background: #444; color: #f0f0f0; }
    body.dark-mode .food-cat-header { background: #333; }
    body.dark-mode .food-cat-name { color: #ccc; }
    body.dark-mode .food-cat-body { background: #2a2a2a; }
    body.dark-mode #foodSearchInput { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; }
    body.dark-mode .fda-label { background: #fff; color: #000; } /* keep FDA label white for legibility */
    body.dark-mode .fda-combo-name { color: #f0f0f0; }
    body.dark-mode .fda-combo-sub { color: #aaa; }
    body.dark-mode .alert.warn { background: #3a2800; border-color: #a0620d; }
    body.dark-mode .alert.good { background: #0a2e15; border-color: #2e7d32; }
    body.dark-mode .fav-item { background: #2a2a2a; border-color: #3a3a3a; }
    body.dark-mode .fav-item-name { color: #f0f0f0; }
    body.dark-mode .fav-btn.del { background: #3a1515; }
    body.dark-mode .goal-input { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; }
    body.dark-mode .goal-bar-track { background: #3a3a3a; }
    body.dark-mode .filter-result-item { border-bottom-color: #3a3a3a; }
    body.dark-mode .filter-result-name { color: #f0f0f0; }
    body.dark-mode .compare-sel select { background: #2a2a2a; color: #f0f0f0; }
    body.dark-mode .empty-state { background: #1a1a1a; }
    body.dark-mode .empty-state p { color: #aaa; }
    body.dark-mode .size-toggle { background: #333; }
    body.dark-mode .size-btn.active { background: #444; color: #f0f0f0; }
    body.dark-mode .macro-chart-wrap { background: #333; }
    body.dark-mode .macro-legend-item span:first-child { color: #f0f0f0; }
  ```

  Replace with the full expanded block (from index-por.html, lines 38–180, plus the legal-panel box-shadow merged in):

  ```css
    body.dark-mode {
      --cream: #1e1e1e;
      --white: #2a2a2a;
      --light: #3a3a3a;
      --mid: #aaa;
      --dark-text: #f0f0f0;
      --surface: #2a2a2a;
      --surface2: #1e1e1e;
      --border: #3a3a3a;
      --text: #f0f0f0;
      --text-mid: #aaa;
    }

    body.dark-mode .scroll-area { background: #1a1a1a; }
    body.dark-mode .card { background: #2a2a2a; box-shadow: 0 2px 12px rgba(0,0,0,0.3); }
    body.dark-mode select { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2020/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%23aaa' stroke-width='2' fill='none' stroke-linecap='round'/%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 12px center; }
    body.dark-mode .smoothie-preview { background: #333; }
    body.dark-mode .addin-card { background: #333; border-color: #444; }
    body.dark-mode .addin-name { color: #f0f0f0; }
    body.dark-mode .food-toggle { background: #333; }
    body.dark-mode .food-toggle-btn.active { background: #444; color: #f0f0f0; }
    body.dark-mode .food-cat-header { background: #333; }
    body.dark-mode .food-cat-name { color: #ccc; }
    body.dark-mode .food-cat-body { background: #2a2a2a; }
    body.dark-mode #foodSearchInput { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; }
    body.dark-mode .fda-label { background: #fff; color: #000; } /* keep FDA label white for legibility */
    body.dark-mode .fda-combo-name { color: #f0f0f0; }
    body.dark-mode .fda-combo-sub { color: #aaa; }
    body.dark-mode .alert.warn { background: #3a2800; border-color: #a0620d; }
    body.dark-mode .alert.good { background: #0a2e15; border-color: #2e7d32; }
    body.dark-mode .fav-item { background: #2a2a2a; border-color: #3a3a3a; }
    body.dark-mode .fav-item-name { color: #f0f0f0; }
    body.dark-mode .fav-btn.del { background: #3a1515; }
    body.dark-mode .goal-input { background: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; }
    body.dark-mode .goal-bar-track { background: #3a3a3a; }
    body.dark-mode .filter-result-item { border-bottom-color: #3a3a3a; }
    body.dark-mode .filter-result-name { color: #f0f0f0; }
    body.dark-mode .compare-sel select { background: #2a2a2a; color: #f0f0f0; }
    body.dark-mode .empty-state { background: #1a1a1a; }
    body.dark-mode .empty-state p { color: #aaa; }
    body.dark-mode .size-toggle { background: #333; }
    body.dark-mode .size-btn.active { background: #444; color: #f0f0f0; }
    body.dark-mode .macro-chart-wrap { background: #333; }
    body.dark-mode .macro-legend-item span:first-child { color: #f0f0f0; }

    /* ── Dark mode — expanded overrides (from POR) ── */

    /* Legal panels */
    body.dark-mode .legal-panel { background: #2a2a2a; box-shadow: 0 2px 12px rgba(0,0,0,0.3); }
    body.dark-mode .legal-header { background: #2a2a2a; border-bottom-color: #3a3a3a; }
    body.dark-mode .legal-header-left { color: #f0f0f0; }
    body.dark-mode .legal-body { color: #aaa; }
    body.dark-mode .legal-body strong { color: #f0f0f0; }
    body.dark-mode .legal-effective { color: #666; }

    /* About tab */
    body.dark-mode .about-row { border-bottom-color: #2a2a2a; }
    body.dark-mode .about-row .lbl { color: #777; }
    body.dark-mode .about-row .val { color: #f0f0f0; }
    body.dark-mode .about-note { color: #777; }

    /* Compare tab */
    body.dark-mode .compare-side { background: #2a2a2a; }
    body.dark-mode .compare-results { background: #2a2a2a; }
    body.dark-mode .compare-row { border-bottom-color: #3a3a3a; }
    body.dark-mode .compare-row.header { background: #222; }
    body.dark-mode .compare-cell { color: #f0f0f0; }
    body.dark-mode .compare-cell.nutrient { color: #888; }
    body.dark-mode .compare-winner-banner.a { background: #0d2a4a; color: #90caf9; }
    body.dark-mode .compare-winner-banner.b { background: #2a0d3a; color: #ce93d8; }
    body.dark-mode .compare-cell.winner-a { color: #90caf9; }
    body.dark-mode .compare-cell.winner-b { color: #ce93d8; }
    body.dark-mode .compare-addin-chip.sel-a { border-color: #1565c0; background: #0d2a4a; color: #90caf9; }
    body.dark-mode .compare-addin-chip.sel-b { border-color: #6a1b9a; background: #2a0d3a; color: #ce93d8; }
    body.dark-mode .addin-card.food-selected { border-color: #9c4dcc; background: #2a1a3a; }
    body.dark-mode .addin-card.food-selected::after { color: #ce93d8; }
    body.dark-mode .compare-addin-chip { background: #2a2a2a; border-color: #3a3a3a; color: #aaa; }

    /* Food categories */
    body.dark-mode .food-cat-divider { background: #3a3a3a; }
    body.dark-mode .food-cat-arrow { color: #aaa; }
    body.dark-mode .addin-cal { color: #888; }
    body.dark-mode .food-items { background: #1e1e1e; }

    /* Section labels */
    body.dark-mode .section-label { color: #666; }
    body.dark-mode .section-label::after { background: #2a2a2a; }

    /* Buttons */
    body.dark-mode .btn-secondary { color: #888; border-color: #3a3a3a; }
    body.dark-mode .btn-secondary:active { border-color: var(--red); color: var(--red); }

    /* Goal mode */
    body.dark-mode .goal-input-label { color: #888; }
    body.dark-mode .goal-bar-name { color: #f0f0f0; }
    body.dark-mode .goal-bar-vals { color: #888; }
    body.dark-mode .goal-summary { background: #222; color: #aaa; }
    body.dark-mode #goalProgressCard { background: #2a2a2a; }

    /* Favorites */
    body.dark-mode .fav-item-sub { color: #888; }
    body.dark-mode .fav-macro { color: #888; }
    body.dark-mode .fav-macro strong { color: #f0f0f0; }
    body.dark-mode .fav-btn.load { background: var(--teal); }
    body.dark-mode .fav-empty { color: #666; }

    /* Filter */
    body.dark-mode .filter-empty { color: #666; }
    body.dark-mode .filter-result-cat { color: #888; }
    body.dark-mode .filter-stat { color: #888; }
    body.dark-mode .filter-stat strong { color: #f0f0f0; }
    body.dark-mode .per100-label { color: #888; }
    body.dark-mode .filter-stat.highlight strong { color: #4db6ac; }
    body.dark-mode .toggle-track { background: #444; }
    body.dark-mode .toggle-track::before { background: #ccc; }

    /* Size toggle */
    body.dark-mode .size-label { color: #777; }
    body.dark-mode .size-btn { color: #777; }

    /* Smoothie preview */
    body.dark-mode .preview-macro { color: #aaa; }
    body.dark-mode .preview-macro strong { color: #f0f0f0; }
    body.dark-mode .preview-cal-label { color: #888; }

    /* Tags */
    body.dark-mode .selected-tags .tag { opacity: 0.9; }

    /* Alerts */
    body.dark-mode .alert strong { color: inherit; }

    /* Toast */
    body.dark-mode .toast { background: #f0f0f0; color: #1a1a1a; }

    /* Field labels */
    body.dark-mode .field-label { color: #777; }

    /* Macro chart legend */
    body.dark-mode .macro-legend-name { color: #f0f0f0; }
    body.dark-mode .macro-legend-val { color: #888; }
    body.dark-mode .macro-chart-title { color: #777; }
    body.dark-mode .macro-donut-lbl { color: #888; }
  ```

- [ ] **Step 2: Remove the now-redundant scattered dark mode rules**

  Remove line 370 (the old `.filter-goal-select` dark override — the select element is being removed):
  ```css
  body.dark-mode .filter-goal-select { background-color: #2a2a2a; color: #f0f0f0; border-color: #3a3a3a; }
  ```

  Remove line 424 (the standalone `.legal-panel` box-shadow rule — now merged into the expanded block above):
  ```css
  body.dark-mode .legal-panel { box-shadow: 0 2px 12px rgba(0,0,0,0.3); }
  ```

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "feat: expand dark mode override block to full POR coverage (v3.1)"
  ```

---

## Chunk 2: HIG Accordion CSS

### Task 2: Add accordion + goal row CSS to the `<style>` block

**Files:**
- Modify: `index.html` — insert new CSS section just before the closing `</style>` tag (currently line 425)

- [ ] **Step 1: Insert the following CSS block** immediately before `</style>` (after the legal panel rules):

  ```css
  /* ── HIG Filter Accordion ── */
  .filter-accordion { display: flex; flex-direction: column; gap: 0; margin-bottom: 14px; border-radius: 12px; overflow: hidden; border: 0.5px solid var(--light); }

  .acc-group {}

  .acc-header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 11px 14px; background: var(--white); cursor: pointer;
    user-select: none; border-bottom: 0.5px solid var(--light);
    gap: 8px;
  }
  .acc-group:last-child .acc-header { border-bottom: none; }
  .acc-group.open .acc-header { border-bottom: 0.5px solid var(--light); }

  .acc-header-left { display: flex; align-items: center; gap: 8px; flex: 1; }
  .acc-header-name {
    font-size: 11px; font-weight: 700; letter-spacing: 0.8px;
    text-transform: uppercase; color: var(--mid);
  }
  .acc-badge {
    display: none; background: var(--teal); color: #fff; font-size: 10px;
    font-weight: 700; padding: 2px 8px; border-radius: 20px; white-space: nowrap;
  }
  .acc-badge.visible { display: inline-block; }

  .acc-chevron {
    font-size: 11px; color: var(--mid); transition: transform 0.2s;
    display: inline-block;
  }
  .acc-group.open .acc-chevron { transform: rotate(180deg); }

  .acc-body { display: none; }
  .acc-group.open .acc-body { display: block; }

  .goal-row {
    display: flex; align-items: center; gap: 12px;
    padding: 11px 14px; background: var(--white); cursor: pointer;
    border-bottom: 0.5px solid var(--light);
    transition: background 0.15s;
  }
  .acc-body .goal-row:last-child { border-bottom: none; }
  .goal-row:active { background: #f0faf9; }
  .goal-row.selected { background: #e8f5f3; }

  .goal-radio {
    width: 20px; height: 20px; border-radius: 50%;
    border: 2px solid var(--light); background: var(--white);
    flex-shrink: 0; display: flex; align-items: center; justify-content: center;
    transition: all 0.15s;
  }
  .goal-row.selected .goal-radio {
    background: var(--teal); border-color: var(--teal);
  }
  .goal-radio-check {
    display: none; color: #fff; font-size: 11px; font-weight: 700; line-height: 1;
  }
  .goal-row.selected .goal-radio-check { display: block; }

  .goal-row-text { flex: 1; }
  .goal-row-name { font-size: 13px; font-weight: 700; color: var(--dark); }
  .goal-row-desc { font-size: 11px; color: var(--mid); margin-top: 1px; line-height: 1.4; }

  /* Per-100-cal row inside filter card */
  .filter-per100-row {
    display: flex; align-items: center; justify-content: space-between;
    padding: 10px 2px; margin-bottom: 10px;
  }
  .filter-per100-row .per100-label {
    font-size: 13px; font-weight: 500; color: var(--dark); letter-spacing: 0; text-transform: none;
  }

  /* Dark mode — accordion */
  body.dark-mode .acc-header { background: #2a2a2a; border-bottom-color: #3a3a3a; }
  body.dark-mode .acc-header-name { color: #888; }
  body.dark-mode .acc-group.open .acc-header { border-bottom-color: #3a3a3a; }
  body.dark-mode .filter-accordion { border-color: #3a3a3a; }
  body.dark-mode .goal-row { background: #2a2a2a; border-bottom-color: #3a3a3a; }
  body.dark-mode .goal-row:active { background: #222; }
  body.dark-mode .goal-row.selected { background: #0a2e28; }
  body.dark-mode .goal-radio { border-color: #3a3a3a; background: #2a2a2a; }
  body.dark-mode .goal-row.selected .goal-radio { background: var(--teal); border-color: var(--teal); }
  body.dark-mode .goal-row-name { color: #f0f0f0; }
  body.dark-mode .goal-row-desc { color: #aaa; }
  body.dark-mode .filter-per100-row .per100-label { color: #f0f0f0; }
  ```

- [ ] **Step 2: Commit**
  ```bash
  git add index.html
  git commit -m "feat: add HIG accordion + goal row CSS for filter tab"
  ```

---

## Chunk 3: Filter Tab HTML Replacement

### Task 3: Replace `<select>` + description with accordion groups

**Files:**
- Modify: `index.html:668–699` (filter card body — goal selector + menu type)

- [ ] **Step 1: Replace the goal selector block** — find this exact HTML in the filter card body:

  ```html
          <p style="font-size:12px;color:var(--mid);margin-bottom:12px;line-height:1.5;">Pick a goal and menu type — top 10 ranked instantly.</p>

          <span class="field-label">Your Goal</span>
          <select class="filter-goal-select" id="filterGoalSelect" onchange="onFilterGoalChange()">
            <optgroup label="── Simple Goals ──">
              <option value="protein">💪 Most Protein</option>
              <option value="cal_low">🔥 Lowest Calories</option>
              <option value="cal_high">🏋️ Highest Calories</option>
              <option value="fiber">🌿 Most Fiber</option>
              <option value="sugar_low">🚫 Lowest Sugar</option>
              <option value="sodium_low">🧂 Lowest Sodium</option>
              <option value="fat_low">🥗 Lowest Fat</option>
            </optgroup>
            <optgroup label="── Efficiency Ratios ──">
              <option value="protein_cal">⚡ Best Protein/Cal Ratio</option>
              <option value="protein_carb">🧬 Best Protein/Carb Ratio</option>
              <option value="fiber_cal">🌾 Best Fiber/Cal Ratio</option>
              <option value="sugar_cal">🩸 Lowest Sugar/Cal Ratio</option>
              <option value="fat_cal">💧 Lowest Fat/Cal Ratio</option>
            </optgroup>
            <optgroup label="── Composite ──">
              <option value="clean_score">🏆 Best "Clean Meal" Score</option>
            </optgroup>
          </select>
          <div class="filter-goal-desc" id="filterGoalDesc"></div>

          <span class="field-label">Menu Type</span>
          <div class="filter-type-row" id="filterTypeRow">
            <div class="filter-type-btn active" data-type="smoothies" onclick="setFilterType(this)">🥤 Smoothies</div>
            <div class="filter-type-btn" data-type="food" onclick="setFilterType(this)">🍽️ Food</div>
            <div class="filter-type-btn" data-type="both" onclick="setFilterType(this)">🔀 Both</div>
          </div>
  ```

  Replace with:

  ```html
          <p style="font-size:12px;color:var(--mid);margin-bottom:12px;line-height:1.5;">Pick a goal and menu type — top 10 ranked instantly.</p>

          <span class="field-label">Your Goal</span>
          <div class="filter-accordion" id="filterAccordion">

            <!-- Group 1: Simple Goals -->
            <div class="acc-group open" id="accGroup-simple">
              <div class="acc-header" onclick="toggleAccGroup('simple')">
                <div class="acc-header-left">
                  <span class="acc-header-name">Simple Goals</span>
                  <span class="acc-badge" id="accBadge-simple"></span>
                </div>
                <span class="acc-chevron">▾</span>
              </div>
              <div class="acc-body">
                <div class="goal-row selected" data-goal="protein" onclick="selectGoalRow('protein')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">💪 Most Protein</div>
                    <div class="goal-row-desc">Ranked by total protein grams</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="cal_low" onclick="selectGoalRow('cal_low')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🔥 Lowest Calories</div>
                    <div class="goal-row-desc">Fewest calories — calorie-controlled diets</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="cal_high" onclick="selectGoalRow('cal_high')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🏋️ Highest Calories</div>
                    <div class="goal-row-desc">Most calories — bulking or high-energy days</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="fiber" onclick="selectGoalRow('fiber')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🌿 Most Fiber</div>
                    <div class="goal-row-desc">Supports digestion, keeps you fuller longer</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="sugar_low" onclick="selectGoalRow('sugar_low')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🚫 Lowest Sugar</div>
                    <div class="goal-row-desc">Fewest sugar grams — blood sugar management</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="sodium_low" onclick="selectGoalRow('sodium_low')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🧂 Lowest Sodium</div>
                    <div class="goal-row-desc">Least sodium — heart health and blood pressure</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="fat_low" onclick="selectGoalRow('fat_low')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🥗 Lowest Fat</div>
                    <div class="goal-row-desc">Fewest fat grams — low-fat dietary goals</div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Group 2: Efficiency Ratios -->
            <div class="acc-group open" id="accGroup-ratios">
              <div class="acc-header" onclick="toggleAccGroup('ratios')">
                <div class="acc-header-left">
                  <span class="acc-header-name">Efficiency Ratios</span>
                  <span class="acc-badge" id="accBadge-ratios"></span>
                </div>
                <span class="acc-chevron">▾</span>
              </div>
              <div class="acc-body">
                <div class="goal-row" data-goal="protein_cal" onclick="selectGoalRow('protein_cal')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">⚡ Best Protein/Cal Ratio</div>
                    <div class="goal-row-desc">Grams of protein per 100 calories</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="protein_carb" onclick="selectGoalRow('protein_carb')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🧬 Best Protein/Carb Ratio</div>
                    <div class="goal-row-desc">Protein ÷ carbs — low-carb and keto-adjacent</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="fiber_cal" onclick="selectGoalRow('fiber_cal')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🌾 Best Fiber/Cal Ratio</div>
                    <div class="goal-row-desc">Fiber grams per 100 calories</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="sugar_cal" onclick="selectGoalRow('sugar_cal')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🩸 Lowest Sugar/Cal Ratio</div>
                    <div class="goal-row-desc">Sugar as % of total calories</div>
                  </div>
                </div>
                <div class="goal-row" data-goal="fat_cal" onclick="selectGoalRow('fat_cal')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">💧 Lowest Fat/Cal Ratio</div>
                    <div class="goal-row-desc">Fat as % of total calories</div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Group 3: Composite -->
            <div class="acc-group open" id="accGroup-composite">
              <div class="acc-header" onclick="toggleAccGroup('composite')">
                <div class="acc-header-left">
                  <span class="acc-header-name">Composite</span>
                  <span class="acc-badge" id="accBadge-composite"></span>
                </div>
                <span class="acc-chevron">▾</span>
              </div>
              <div class="acc-body">
                <div class="goal-row" data-goal="clean_score" onclick="selectGoalRow('clean_score')">
                  <div class="goal-radio"><span class="goal-radio-check">✓</span></div>
                  <div class="goal-row-text">
                    <div class="goal-row-name">🏆 Best "Clean Meal" Score</div>
                    <div class="goal-row-desc">Weighted composite: protein, fiber, sugar, sodium, fat</div>
                  </div>
                </div>
              </div>
            </div>

          </div><!-- /.filter-accordion -->

          <!-- Per-100-cal toggle (simple goals only) -->
          <div class="filter-per100-row" id="filterPer100Row">
            <span class="per100-label">Per 100 cal</span>
            <label class="toggle-switch">
              <input type="checkbox" id="per100Toggle" onchange="runFilter()">
              <div class="toggle-track"></div>
            </label>
          </div>

          <span class="field-label">Menu Type</span>
          <div class="filter-type-row" id="filterTypeRow">
            <div class="filter-type-btn active" data-type="smoothies" onclick="setFilterType(this)">🥤 Smoothies</div>
            <div class="filter-type-btn" data-type="food" onclick="setFilterType(this)">🍽️ Food</div>
            <div class="filter-type-btn" data-type="both" onclick="setFilterType(this)">🔀 Both</div>
          </div>
  ```

  **Note:** The per-100-cal toggle has been moved here from the results card header (lines ~708–714). Also remove the now-redundant per-100-cal toggle from the results card header. Find and remove this block from the results card:
  ```html
          <div class="per100-toggle" style="margin-left:auto;">
            <span class="per100-label">Per 100 cal</span>
            <label class="toggle-switch">
              <input type="checkbox" id="per100Toggle" onchange="runFilter()">
              <div class="toggle-track"></div>
            </label>
          </div>
  ```

- [ ] **Step 2: Commit**
  ```bash
  git add index.html
  git commit -m "feat: replace filter goal select with HIG accordion UI"
  ```

---

## Chunk 4: Filter JS Updates

### Task 4: Update JS for accordion behavior and per-100-cal visibility

**Files:**
- Modify: `index.html` — JS section (~lines 1819–1823 and ~1960)

- [ ] **Step 1: Replace `onFilterGoalChange()`** — find:

  ```js
  function onFilterGoalChange() {
    filterGoal = document.getElementById('filterGoalSelect').value;
    const cfg = GOAL_CONFIG[filterGoal];
    document.getElementById('filterGoalDesc').textContent = cfg.desc;
    runFilter();
  }
  ```

  Replace with:

  ```js
  const SIMPLE_GOALS = ['protein','cal_low','cal_high','fiber','sugar_low','sodium_low','fat_low'];

  // Map goal value → accordion group ID
  const GOAL_GROUP = {
    protein:'simple', cal_low:'simple', cal_high:'simple', fiber:'simple',
    sugar_low:'simple', sodium_low:'simple', fat_low:'simple',
    protein_cal:'ratios', protein_carb:'ratios', fiber_cal:'ratios',
    sugar_cal:'ratios', fat_cal:'ratios',
    clean_score:'composite'
  };

  function selectGoalRow(value) {
    // Deselect all rows, select the tapped one
    document.querySelectorAll('#filterAccordion .goal-row').forEach(r => r.classList.remove('selected'));
    const row = document.querySelector('#filterAccordion .goal-row[data-goal="' + value + '"]');
    if (row) row.classList.add('selected');
    filterGoal = value;
    updateAccordionBadges();
    updatePer100Visibility();
    runFilter();
  }

  function toggleAccGroup(groupId) {
    const groups = ['simple','ratios','composite'];
    groups.forEach(id => {
      const el = document.getElementById('accGroup-' + id);
      if (id === groupId) {
        el.classList.toggle('open');
      } else {
        el.classList.remove('open');
      }
    });
    updateAccordionBadges();
  }

  function updateAccordionBadges() {
    ['simple','ratios','composite'].forEach(id => {
      const group = document.getElementById('accGroup-' + id);
      const badge = document.getElementById('accBadge-' + id);
      if (!group || !badge) return;
      const isOpen = group.classList.contains('open');
      const selectedRow = group.querySelector('.goal-row.selected');
      if (!isOpen && selectedRow) {
        badge.textContent = selectedRow.querySelector('.goal-row-name').textContent;
        badge.classList.add('visible');
      } else {
        badge.classList.remove('visible');
      }
    });
  }

  function updatePer100Visibility() {
    const row = document.getElementById('filterPer100Row');
    if (!row) return;
    row.style.display = SIMPLE_GOALS.includes(filterGoal) ? 'flex' : 'none';
  }

  function onFilterGoalChange() {
    // Legacy shim — no longer called directly, kept for safety
    runFilter();
  }
  ```

- [ ] **Step 2: Remove the `filterGoalDesc` init line** — find and delete:

  ```js
  document.getElementById('filterGoalDesc').textContent = GOAL_CONFIG['protein'].desc;
  ```

  Also add `updatePer100Visibility();` to the init block immediately after `runFilter();`:
  ```js
  runFilter();
  updatePer100Visibility();
  ```

- [ ] **Step 3: Commit**
  ```bash
  git add index.html
  git commit -m "feat: add accordion JS — row select, group toggle, per-100-cal visibility"
  ```

---

## Chunk 5: Version Bump + Cleanup

### Task 5: Bump version badge and delete index-por.html

**Files:**
- Modify: `index.html:436`
- Delete: `index-por.html`

- [ ] **Step 1: Bump version badge** — find:
  ```html
      <div class="header-badge">🥤 v3.0</div>
  ```
  Replace with:
  ```html
      <div class="header-badge">🥤 v3.1</div>
  ```

- [ ] **Step 2: Delete index-por.html**
  ```bash
  rm /c/Users/clark/OneDrive/Documents/GitHub/tsc-calc/index-por.html
  ```

- [ ] **Step 3: Final commit**
  ```bash
  git add index.html
  git rm index-por.html
  git commit -m "chore: bump to v3.1, remove POR reference file"
  ```

---

## Manual Smoke Test Checklist

After all tasks complete, open `index.html` in a browser and verify:

- [ ] Dark mode toggle works — all tabs look correct in both light and dark
- [ ] Legal panels in About tab open/close correctly in both light and dark
- [ ] Filter tab shows accordion with 3 groups (all expanded by default)
- [ ] Tapping a goal row selects it (teal fill radio, tinted row background)
- [ ] Tapping a group header collapses/expands it (chevron rotates)
- [ ] Only one group can be open at a time
- [ ] When a group is collapsed with an active selection, teal pill badge appears in header
- [ ] Per-100-cal toggle visible when Simple Goal selected, hidden for Ratios/Composite
- [ ] Per-100-cal toggle still works (results change when toggled on Simple Goals)
- [ ] Menu type toggle (Smoothies/Food/Both) still works
- [ ] Results card still shows ranked items
- [ ] Version badge shows v3.1
- [ ] No other tabs (Builder, Label, Saved, Goals, Compare, About) are affected
