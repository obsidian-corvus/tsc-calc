# Legal Pages Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Privacy Policy, Terms of Service, and Disclaimer as accordion panels inside the existing About tab of `index.html`.

**Architecture:** Three new `.legal-panel` cards appended inside `#view-about` (after the existing "About This App" card). A single `toggleLegal(id)` function handles accordion behavior — opening one panel closes the others. All styles use existing CSS variables so dark mode works automatically. No new files, no new dependencies.

**Tech Stack:** Vanilla HTML/CSS/JS, inline in `index.html`. No build tools.

> **Note on TDD:** No unit tests are written for this feature. The implementation is pure DOM/CSS manipulation in a no-build single-file app — there is no testable business logic to isolate. Verification is done manually in the browser per the checklist steps.

---

## Chunk 1: CSS + JS

### Task 1: Add legal panel CSS to the style block

**Files:**
- Modify: `index.html` (style block, before line 319 `</style>`)

- [ ] **Step 1: Read the file**

  Open `index.html` and confirm line 319 is `</style>`.

- [ ] **Step 2: Insert CSS before `</style>`**

  Add the following block immediately before `</style>` (line 319):

  ```css
  /* ── Legal Panels ── */
  .legal-panel { background: var(--white); border-radius: 14px; overflow: hidden; margin-bottom: 14px; box-shadow: 0 2px 12px rgba(0,0,0,0.07); }
  .legal-header { padding: 14px 16px; display: flex; align-items: center; justify-content: space-between; cursor: pointer; user-select: none; gap: 10px; }
  .legal-header-left { display: flex; align-items: center; gap: 10px; font-family: 'Bebas Neue', sans-serif; font-size: 17px; letter-spacing: 1px; color: var(--dark); }
  .legal-chevron { font-size: 12px; color: var(--mid); transition: transform 0.2s; display: inline-block; }
  .legal-chevron.open { transform: rotate(90deg); }
  .legal-body { display: none; padding: 0 16px 16px; font-size: 13px; color: var(--mid); line-height: 1.7; }
  .legal-body.open { display: block; }
  .legal-body ul { margin: 8px 0 0 0; padding-left: 18px; }
  .legal-body ul li { margin-bottom: 6px; }
  .legal-effective { font-size: 11px; font-weight: 700; letter-spacing: 1px; text-transform: uppercase; color: var(--mid); margin-bottom: 10px; display: block; }
  ```

- [ ] **Step 3: Manually verify in browser**

  Serve `index.html` locally (VS Code Live Server on port 5500, or `python -m http.server 5500` in cmd, or `npx serve -p 5500` in cmd) and navigate to the About tab. The existing card should still look identical — no visual change yet (classes aren't used yet).

- [ ] **Step 4: Commit**

  ```bash
  git add index.html
  git commit -m "style: add legal panel CSS classes"
  ```

---

### Task 2: Add `toggleLegal()` function to the script block

**Files:**
- Modify: `index.html` (script block, before line 1591 `</script>`)

- [ ] **Step 1: Insert function before `</script>`**

  Add the following immediately before `</script>` (line 1591):

  ```js
  // ── Legal Accordion ──
  function toggleLegal(id) {
    const allBodies = document.querySelectorAll('.legal-body');
    const allChevrons = document.querySelectorAll('.legal-chevron');
    const target = document.getElementById(id);
    const isOpen = target.classList.contains('open');
    allBodies.forEach(b => b.classList.remove('open'));
    allChevrons.forEach(c => c.classList.remove('open'));
    if (!isOpen) {
      target.classList.add('open');
      target.previousElementSibling.querySelector('.legal-chevron').classList.add('open');
    }
  }
  ```

- [ ] **Step 2: Open browser console and smoke-test the function**

  Open `http://localhost:5500`, go to Console tab, and run:
  ```js
  typeof toggleLegal
  ```
  Expected output: `"function"`

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add toggleLegal accordion function"
  ```

---

## Chunk 2: HTML Panels

### Task 3: Add Privacy Policy panel

**Files:**
- Modify: `index.html` (inside `#view-about`, after the closing `</div>` of the existing About card — after line 657)

- [ ] **Step 1: Insert Privacy Policy panel**

  Locate the closing `</div>` of the "About This App" card (line 657). Insert the following immediately after it (before line 658 `</div>` that closes `#view-about`):

  ```html
  <!-- Privacy Policy -->
  <div class="legal-panel">
    <div class="legal-header" onclick="toggleLegal('legal-privacy')">
      <span class="legal-header-left">🔒 Privacy Policy</span>
      <span class="legal-chevron" id="chev-privacy">▶</span>
    </div>
    <div class="legal-body" id="legal-privacy">
      <span class="legal-effective">Effective: March 14, 2026</span>
      <ul>
        <li>No personal data is collected or transmitted to external servers during normal use.</li>
        <li>All user data (saved orders, favorites, goals) is stored exclusively in <strong>localStorage</strong> on your device.</li>
        <li>No cookies, no third-party analytics, no tracking.</li>
        <li>The AI analysis feature routes requests through a secure proxy server. No user data is logged or retained by the proxy.</li>
        <li>Data practices may change as features expand. This policy will be updated accordingly.</li>
      </ul>
    </div>
  </div>
  ```

  **Note on chevron IDs:** The `id="chev-privacy"` (and `id="chev-tos"`, `id="chev-disclaimer"` on subsequent panels) are **not used by `toggleLegal()`**. The function targets chevrons via DOM traversal (`target.previousElementSibling.querySelector('.legal-chevron')`), not by ID. These IDs are included purely as debugging handles — you can select them in DevTools console (e.g. `document.getElementById('chev-privacy')`) to inspect state. Do not rewrite the function to use them.

- [ ] **Step 2: Verify in browser**

  Open `http://localhost:5500` → About tab. You should see a "🔒 Privacy Policy" panel below the About card. It should be collapsed. Tap/click the header — it should expand showing the bullet list.

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add Privacy Policy accordion panel"
  ```

---

### Task 4: Add Terms of Service panel

**Files:**
- Modify: `index.html` (inside `#view-about`, after the Privacy Policy panel)

- [ ] **Step 1: Insert Terms of Service panel**

  After the Privacy Policy panel's closing `</div>`, insert:

  ```html
  <!-- Terms of Service -->
  <div class="legal-panel">
    <div class="legal-header" onclick="toggleLegal('legal-tos')">
      <span class="legal-header-left">📋 Terms of Service</span>
      <span class="legal-chevron" id="chev-tos">▶</span>
    </div>
    <div class="legal-body" id="legal-tos">
      <span class="legal-effective">Effective: March 14, 2026</span>
      <ul>
        <li>TSC Nutrition Calculator ("the App") is a product of Obsidian Corvus Strategy LLC.</li>
        <li>The App is provided "as is" without warranties of any kind, express or implied.</li>
        <li>Nutritional data is sourced from restaurant-published materials and is provided for informational purposes only. Values may vary based on preparation and serving size.</li>
        <li>The App is not intended for use in making medical, clinical, or dietary treatment decisions.</li>
        <li>All intellectual property, including the App's design, code, and methodology, is owned by Obsidian Corvus Strategy LLC. Unauthorized reproduction or distribution is prohibited.</li>
        <li>Continued use of the App constitutes acceptance of these terms.</li>
        <li>Terms may be updated at any time. Check this page for the most current version.</li>
      </ul>
    </div>
  </div>
  ```

- [ ] **Step 2: Verify accordion behavior**

  - Open Privacy Policy → it expands ✅
  - Now open Terms of Service → Privacy Policy collapses, ToS expands ✅
  - Tap ToS header again → ToS collapses ✅

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add Terms of Service accordion panel"
  ```

---

### Task 5: Add Disclaimer panel

**Files:**
- Modify: `index.html` (inside `#view-about`, after the Terms of Service panel)

- [ ] **Step 1: Insert Disclaimer panel**

  After the Terms of Service panel's closing `</div>`, insert:

  ```html
  <!-- Disclaimer -->
  <div class="legal-panel">
    <div class="legal-header" onclick="toggleLegal('legal-disclaimer')">
      <span class="legal-header-left">⚠️ Disclaimer</span>
      <span class="legal-chevron" id="chev-disclaimer">▶</span>
    </div>
    <div class="legal-body" id="legal-disclaimer">
      <span class="legal-effective">Effective: March 14, 2026</span>
      <ul>
        <li>Nutritional information displayed in this App is for general informational purposes only.</li>
        <li>Values are based on standard serving sizes as published by the restaurant and may differ from actual prepared items.</li>
        <li>This App is not a substitute for professional nutrition, dietary, or medical advice.</li>
        <li>Always consult the in-store nutrition guide and a qualified professional for decisions affecting your health.</li>
        <li>Obsidian Corvus Strategy LLC makes no representations as to the accuracy or completeness of any information on this App.</li>
      </ul>
    </div>
  </div>
  ```

- [ ] **Step 2: Full accordion test**

  Go through the full testing checklist from the spec:
  - [ ] All three panels start collapsed on page load
  - [ ] Tapping a header expands its panel
  - [ ] Tapping an open header collapses it
  - [ ] Opening one panel collapses any other open panel
  - [ ] Chevron rotates correctly on open/close
  - [ ] Panels render correctly in light mode
  - [ ] No JS errors in console

- [ ] **Step 3: Commit**

  ```bash
  git add index.html
  git commit -m "feat: add Disclaimer accordion panel"
  ```

---

## Chunk 3: Polish + Branch Completion

### Task 6: Dark mode spot-check and branch finish

- [ ] **Step 1: Verify dark mode**

  The panels use `var(--white)`, `var(--dark)`, `var(--mid)` — they inherit dark mode automatically if the app applies a class or CSS variable swap at the root level. Check by toggling dark mode in the app and confirming the panels remain readable.

- [ ] **Step 2: Mobile viewport spot-check**

  Resize the browser window to ~375px wide (iPhone SE width) or use DevTools device emulation. Confirm:
  - Panel text doesn't overflow or get clipped
  - Tap targets (headers) are comfortably large

- [ ] **Step 3: Invoke finishing-a-development-branch skill**

  Use `superpowers:finishing-a-development-branch` to complete the branch and merge.
