# TSC Nutrition Calculator — Claude Code Context

## Project Overview
Restaurant Nutrition Calculator PWA built under **Obsidian Corvus Strategy LLC** (Florida LLC).
- Live: https://obsidian-corvus.github.io/tsc-calc/
- GitHub: https://github.com/obsidian-corvus/tsc-calc
- IP owned by OCS LLC; provisional patent grace period starts March 7, 2026

## Tech Stack
- **Frontend**: Vanilla HTML/CSS/JS — single file (`index.html`, ~1,900 lines). No framework intentional.
- **PWA**: Offline-first via Service Worker (`sw.js`), installable
- **Fonts**: Bebas Neue + DM Sans (Google Fonts CDN)
- **Share card**: html2canvas v1.4.1 (CDN)
- **Backend (planned)**: Node.js proxy on Railway — keeps Anthropic API key off client

## Current Data
All TSC nutrition data hardcoded in `index.html` as JS arrays: `smoothies[]`, `foodCategories[]`, `supplements[]`, `freshAddins[]`.
Each item: `name, cal, fat, satFat, trans, chol, sodium, carb, fiber, sugar, protein`
Source: TSC Nutrition Guide dated 11/06/25.

## Feature Status (v3.0 — complete)
Builder, Smoothie Size Toggle, FDA Label, Macro Donut Chart, Favorites/Saved Orders, Goal Mode, Smart Filter (13 goals + Clean Meal Score), Per-100-Cal Toggle, Context Flags, Compare Mode, Dark Mode, Share Card (html2canvas)

## Roadmap Priorities
### Round 2 (active)
1. **Backend proxy** — Node.js on Railway, proxies Anthropic API, keeps key secure
2. **PDF Upload + Claude Parsing** — user uploads restaurant PDF → backend parses → structured menu data
3. **Multi-restaurant UI** — TSC becomes first entry; restaurant switcher

### Round 3
- Order History, Build-to-Goal Mode, Streak Tracker
- **Legal Pages** — Privacy Policy, ToS, Disclaimer in About tab (expandable panels)
- **Micronutrients** — vitamins + minerals from USDA FoodData Central where available

### Parallel (non-blocking)
- MUI / Apple HIG UI prototype on separate branch

## Architecture Decisions
- Vanilla JS over React — single-file, offline-first, zero build step
- Railway over Render — no cold starts on free tier ($5/mo credit)
- PDF upload over DB integration — no partnerships, works with public data
- B2B primary monetization — direct restaurant chain pitches
- FDA label stays white in dark mode — legibility priority
- Per-100-cal toggle: simple goals only (ratio goals already normalize by cal)
- NEVER estimate nutritional data — show "N/A" if unavailable

## Backend Proxy Spec (next build)
- Separate GitHub repo: `ocsc-proxy`
- ~50 lines Node.js (Express)
- Endpoint: POST /api/claude → forwards to Anthropic API
- API key in Railway env var: `ANTHROPIC_API_KEY`
- Returns Anthropic response to frontend
- Later: POST /api/parse-pdf for PDF text extraction

## Legal Pages Requirements (Section 9 of brief)
- Live inside About tab as expandable panels (not separate pages)
- **Privacy Policy**: localStorage only, no server data currently, note future proxy may log anonymized usage
- **ToS**: "as is", no medical reliance, IP owned by OCS LLC, data from restaurant-published materials
- **Disclaimer**: informational only, not medical advice, values may vary

## USDA FoodData Central
- API: https://fdc.nal.usda.gov/food-search
- Free, no auth for basic queries
- Use for micronutrient gaps only — always label as "USDA estimate"
- Never interpolate; show "N/A" when data unavailable
- Micronutrients to add: Vitamin A/C/D/E/K, B1/B2/B3/B6/B12, Folate, Calcium, Iron, Potassium, Magnesium, Zinc, Phosphorus, Selenium, Copper, Manganese

## Code Style
- Inline styles in `<style>` block at top of index.html
- CSS variables: `--teal`, `--light`, `--mid`, etc.
- Tab switching via `switchTab(name)` function
- No build tools, no transpilation — pure browser JS
- Emoji icons used throughout UI intentionally

## Development Notes
- Do not block Round 2 on MUI exploration
- Do not remove existing features
- Single-file constraint is intentional — preserve it unless multi-restaurant rebuild starts
