# Backend Proxy (ocsc-proxy) Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and deploy a lightweight Node.js Express proxy on Railway that forwards requests from the TSC Calc frontend to the Anthropic API, keeping the API key off the client.

**Architecture:** A single Express server (`index.js`) validates an `X-Proxy-Token` header, injects the `ANTHROPIC_API_KEY` and required Anthropic headers, then forwards the request body verbatim to `api.anthropic.com/v1/messages`. All secrets live in Railway environment variables, never in code or Git.

**Tech Stack:** Node.js 18+, Express 4, cors, Node built-in `fetch` (no node-fetch), Node built-in test runner (`node:test`), supertest

**Spec:** `docs/superpowers/specs/2026-03-14-backend-proxy-design.md`

---

> **Note for new backend developers:** This is a *new, separate repository* — not a change to the existing `tsc-calc` repo. You'll create a new folder called `ocsc-proxy` on your computer, build the proxy there, push it to a new GitHub repo, then connect that GitHub repo to Railway for deployment.

---

## Chunk 1: Project scaffold

### Task 1: Create the repository

**Files:**
- Create: `ocsc-proxy/package.json`
- Create: `ocsc-proxy/.gitignore`
- Create: `ocsc-proxy/.nvmrc`
- Create: `ocsc-proxy/.env.example`

- [ ] **Step 1: Create project directory**

  Pick a location on your computer (e.g. next to your other repos):
  ```bash
  mkdir ocsc-proxy
  cd ocsc-proxy
  git init
  ```

- [ ] **Step 2: Create `package.json`**

  ```json
  {
    "name": "ocsc-proxy",
    "version": "1.0.0",
    "description": "Anthropic API proxy for OCS Nutrition Calculator",
    "main": "index.js",
    "scripts": {
      "start": "node index.js",
      "test": "node --test index.test.js"
    },
    "engines": {
      "node": ">=18"
    },
    "dependencies": {
      "cors": "^2.8.5",
      "express": "^4.18.2"
    },
    "devDependencies": {
      "supertest": "^6.3.4"
    }
  }
  ```

- [ ] **Step 3: Create `.nvmrc`**

  ```
  18
  ```

- [ ] **Step 4: Create `.gitignore`**

  ```
  node_modules/
  .env
  ```

- [ ] **Step 5: Create `.env.example`**

  This is a template — it shows what variables are needed without containing real values.
  Never put real secrets in this file.

  ```
  ANTHROPIC_API_KEY=
  PROXY_TOKEN=
  ```

- [ ] **Step 6: Install dependencies**

  ```bash
  npm install
  ```

  Expected: `node_modules/` folder created, `package-lock.json` created.

- [ ] **Step 7: Commit scaffold**

  ```bash
  git add package.json package-lock.json .gitignore .nvmrc .env.example
  git commit -m "chore: project scaffold"
  ```

---

## Chunk 2: Health endpoint (TDD)

### Task 2: GET /health

**Files:**
- Create: `ocsc-proxy/index.js`
- Create: `ocsc-proxy/index.test.js`

- [ ] **Step 1: Create a minimal `index.js` so the test file can import it**

  ```js
  const express = require('express');
  const app = express();

  app.get('/health', (req, res) => {
    res.json({ status: 'ok' });
  });

  module.exports = app;
  ```

  > We export `app` without calling `app.listen()` here. We'll add the listen call at the bottom later — keeping them separate lets tests import the app without actually starting a server.

- [ ] **Step 2: Write the failing test for `/health`**

  Create `index.test.js`:

  ```js
  const { test } = require('node:test');
  const assert = require('node:assert/strict');
  const request = require('supertest');
  const app = require('./index');

  test('GET /health returns 200 with status ok', async () => {
    const res = await request(app).get('/health');
    assert.equal(res.status, 200);
    assert.deepEqual(res.body, { status: 'ok' });
  });
  ```

- [ ] **Step 3: Run test — expect it to pass**

  ```bash
  npm test
  ```

  Expected output: `✔ GET /health returns 200 with status ok`

- [ ] **Step 4: Commit**

  ```bash
  git add index.js index.test.js
  git commit -m "feat: add /health endpoint"
  ```

---

## Chunk 3: Token auth and /api/claude (TDD)

### Task 3: Token validation middleware

**Files:**
- Modify: `ocsc-proxy/index.js`
- Modify: `ocsc-proxy/index.test.js`

- [ ] **Step 1: Write failing tests for token validation**

  Add to `index.test.js`:

  ```js
  test('POST /api/claude with missing token returns 403', async () => {
    const res = await request(app)
      .post('/api/claude')
      .send({ model: 'claude-haiku-4-5-20251001', max_tokens: 10, messages: [] });
    assert.equal(res.status, 403);
    assert.deepEqual(res.body, { error: 'Forbidden' });
  });

  test('POST /api/claude with wrong token returns 403', async () => {
    const res = await request(app)
      .post('/api/claude')
      .set('X-Proxy-Token', 'wrong-token')
      .send({ model: 'claude-haiku-4-5-20251001', max_tokens: 10, messages: [] });
    assert.equal(res.status, 403);
    assert.deepEqual(res.body, { error: 'Forbidden' });
  });
  ```

- [ ] **Step 2: Run tests — expect the two new tests to fail**

  ```bash
  npm test
  ```

  Expected: 1 pass (health), 2 fail (both 403 tests — endpoint doesn't exist yet)

- [ ] **Step 3: Add token validation and stub `/api/claude` to `index.js`**

  Replace the full contents of `index.js` with:

  ```js
  const express = require('express');
  const cors = require('cors');

  const app = express();

  // --- CORS ---
  const ALLOWED_ORIGINS = [
    'https://mashermike.github.io',
    'http://localhost:3000',
    'http://localhost:5500',
    'http://127.0.0.1:5500',
  ];
  app.use(cors({
    origin: (origin, callback) => {
      // Allow requests with no origin (curl, health checks)
      if (!origin || ALLOWED_ORIGINS.includes(origin)) return callback(null, true);
      callback(new Error('Not allowed by CORS'));
    },
  }));

  // --- Body parsing (50kb limit) ---
  app.use(express.json({ limit: '50kb' }));

  // --- Token validation middleware ---
  function requireToken(req, res, next) {
    const token = req.headers['x-proxy-token'];
    if (!token || token !== process.env.PROXY_TOKEN) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  }

  // --- Routes ---
  app.get('/health', (req, res) => {
    res.json({ status: 'ok' });
  });

  app.post('/api/claude', requireToken, async (req, res) => {
    // Stub — will be implemented in next task
    res.status(501).json({ error: 'Not implemented' });
  });

  module.exports = app;
  ```

- [ ] **Step 4: Run tests — expect 403 tests to pass, health still passes**

  ```bash
  npm test
  ```

  Expected: 3 pass

- [ ] **Step 5: Commit**

  ```bash
  git add index.js index.test.js
  git commit -m "feat: add token validation middleware"
  ```

---

## Chunk 4: Anthropic forwarding (TDD with mocking)

### Task 4: Forward to Anthropic API

**Files:**
- Modify: `ocsc-proxy/index.js`
- Modify: `ocsc-proxy/index.test.js`

- [ ] **Step 1: Write failing tests for Anthropic forwarding**

  > These tests mock the Anthropic API so they don't make real network calls or cost money.

  Add to `index.test.js`:

  ```js
  const { mock } = require('node:test');

  test('POST /api/claude forwards to Anthropic and returns response', async () => {
    // Set env vars for this test
    process.env.PROXY_TOKEN = 'test-token';
    process.env.ANTHROPIC_API_KEY = 'test-key';

    // Mock global fetch
    const mockResponse = { id: 'msg_123', content: [{ text: 'hello' }] };
    mock.method(global, 'fetch', async () => ({
      status: 200,
      headers: { get: () => 'application/json' },
      json: async () => mockResponse,
    }));

    const res = await request(app)
      .post('/api/claude')
      .set('X-Proxy-Token', 'test-token')
      .send({ model: 'claude-haiku-4-5-20251001', max_tokens: 10, messages: [{ role: 'user', content: 'hi' }] });

    assert.equal(res.status, 200);
    assert.deepEqual(res.body, mockResponse);

    // Verify correct headers were sent to Anthropic
    const fetchCall = mock.calls[0];
    // fetchCall.arguments[1].headers should contain Authorization and anthropic-version
    const headers = fetchCall.arguments[1].headers;
    assert.equal(headers['Authorization'], 'Bearer test-key');
    assert.equal(headers['anthropic-version'], '2023-06-01');
    assert.equal(headers['Content-Type'], 'application/json');

    mock.restoreAll();
  });

  test('POST /api/claude passes Anthropic error status through', async () => {
    process.env.PROXY_TOKEN = 'test-token';
    process.env.ANTHROPIC_API_KEY = 'test-key';

    const errorBody = { error: { type: 'invalid_request_error', message: 'bad input' } };
    mock.method(global, 'fetch', async () => ({
      status: 400,
      headers: { get: () => 'application/json' },
      json: async () => errorBody,
    }));

    const res = await request(app)
      .post('/api/claude')
      .set('X-Proxy-Token', 'test-token')
      .send({ model: 'claude-haiku-4-5-20251001', max_tokens: 10, messages: [] });

    assert.equal(res.status, 400);
    assert.deepEqual(res.body, errorBody);

    mock.restoreAll();
  });
  ```

- [ ] **Step 2: Run tests — expect the 2 new forwarding tests to fail**

  ```bash
  npm test
  ```

  Expected: 3 pass, 2 fail (forwarding tests hit the 501 stub)

- [ ] **Step 3: Implement Anthropic forwarding in `index.js`**

  Replace the stub `/api/claude` route with:

  ```js
  app.post('/api/claude', requireToken, async (req, res) => {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 30_000);

    try {
      const upstream = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.ANTHROPIC_API_KEY}`,
          'anthropic-version': '2023-06-01',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(req.body),
        signal: controller.signal,
      });

      // Guard: Anthropic should always return JSON, but parse defensively
      const contentType = upstream.headers.get('content-type') || '';
      const data = contentType.includes('application/json')
        ? await upstream.json()
        : { error: await upstream.text() };
      res.status(upstream.status).json(data);
    } catch (err) {
      if (err.name === 'AbortError') {
        return res.status(504).json({ error: 'Upstream timeout' });
      }
      console.error('Proxy error:', err);
      res.status(500).json({ error: 'Internal Server Error' });
    } finally {
      clearTimeout(timeout);
    }
  });
  ```

- [ ] **Step 4: Run all tests — expect 5 pass**

  ```bash
  npm test
  ```

  Expected: 5 pass, 0 fail

- [ ] **Step 5: Commit**

  ```bash
  git add index.js index.test.js
  git commit -m "feat: forward requests to Anthropic API with timeout"
  ```

---

## Chunk 5: Startup guard and server entry point

### Task 5: Production-ready startup

**Files:**
- Modify: `ocsc-proxy/index.js`

- [ ] **Step 1: Add startup guard and `listen` call to bottom of `index.js`**

  Append to the end of `index.js` (after `module.exports = app;`):

  ```js
  // Only start the server when run directly (not when imported by tests)
  if (require.main === module) {
    const { ANTHROPIC_API_KEY, PROXY_TOKEN, PORT = 3000 } = process.env;
    if (!ANTHROPIC_API_KEY || !PROXY_TOKEN) {
      console.error('ERROR: ANTHROPIC_API_KEY and PROXY_TOKEN must be set.');
      process.exit(1);
    }
    app.listen(PORT, () => {
      console.log(`ocsc-proxy listening on port ${PORT}`);
    });
  }
  ```

  > The `require.main === module` check means: "only run this block if the file was started directly with `node index.js`". When tests import `app`, this block is skipped — so tests don't start a real server on a port.

- [ ] **Step 2: Run tests — still 5 pass**

  ```bash
  npm test
  ```

  Expected: 5 pass — startup block is not triggered by tests.

- [ ] **Step 3: Test startup guard locally**

  ```bash
  # Should fail and exit with error
  node index.js
  ```

  Expected output: `ERROR: ANTHROPIC_API_KEY and PROXY_TOKEN must be set.`

- [ ] **Step 4: Commit**

  ```bash
  git add index.js
  git commit -m "feat: add startup env var guard and server listen"
  ```

---

## Chunk 6: Local smoke test

### Task 6: End-to-end test with real credentials

**Files:**
- Create: `ocsc-proxy/.env` (local only — already in .gitignore)

- [ ] **Step 1: Create your local `.env` file**

  > **Important:** This file must NEVER be committed to Git. It's already in `.gitignore`.

  Create `.env` in the `ocsc-proxy` directory:
  ```
  ANTHROPIC_API_KEY=sk-ant-<your real key>
  PROXY_TOKEN=test-local-token-123
  ```

- [ ] **Step 2: Start the server**

  ```bash
  node -r dotenv/config index.js
  ```

  Wait — `dotenv` isn't installed yet. Use this instead (manually export env vars):

  **On Mac/Linux:**
  ```bash
  export ANTHROPIC_API_KEY=sk-ant-<your key>
  export PROXY_TOKEN=test-local-token-123
  node index.js
  ```

  **On Windows (PowerShell):**
  ```powershell
  $env:ANTHROPIC_API_KEY="sk-ant-<your key>"
  $env:PROXY_TOKEN="test-local-token-123"
  node index.js
  ```

  Expected: `ocsc-proxy listening on port 3000`

- [ ] **Step 3: Test health endpoint**

  Open a new terminal:
  ```bash
  curl http://localhost:3000/health
  ```
  Expected: `{"status":"ok"}`

- [ ] **Step 4: Test auth rejection**

  ```bash
  curl -X POST http://localhost:3000/api/claude \
    -H "Content-Type: application/json" \
    -H "X-Proxy-Token: wrong" \
    -d '{"model":"claude-haiku-4-5-20251001","max_tokens":10,"messages":[{"role":"user","content":"ping"}]}'
  ```
  Expected: `{"error":"Forbidden"}`

- [ ] **Step 5: Test real Anthropic call**

  ```bash
  curl -X POST http://localhost:3000/api/claude \
    -H "Content-Type: application/json" \
    -H "X-Proxy-Token: test-local-token-123" \
    -d '{"model":"claude-haiku-4-5-20251001","max_tokens":10,"messages":[{"role":"user","content":"Say hi"}]}'
  ```
  Expected: JSON response from Anthropic with `"content"` array containing a text response.

- [ ] **Step 6: Stop the server** (`Ctrl+C`)

---

## Chunk 7: README and GitHub push

### Task 7: Documentation and push

**Files:**
- Create: `ocsc-proxy/README.md`

- [ ] **Step 1: Create `README.md`**

  ```markdown
  # ocsc-proxy

  Lightweight Anthropic API proxy for the OCS Nutrition Calculator.
  Deployed on Railway. Keeps the API key off the frontend.

  ## Endpoints

  | Method | Path | Description |
  |--------|------|-------------|
  | GET | /health | Health check |
  | POST | /api/claude | Proxy to Anthropic Messages API |

  ## Authentication

  Every request to `/api/claude` must include:
  ```
  X-Proxy-Token: <value of PROXY_TOKEN env var>
  ```

  ## Environment Variables

  See `.env.example`. Set these in Railway — never commit real values.

  | Variable | Description |
  |----------|-------------|
  | `ANTHROPIC_API_KEY` | Your Anthropic API key |
  | `PROXY_TOKEN` | Shared secret validated on every request |

  ## Local Development

  ```bash
  npm install
  # Set env vars (see .env.example)
  node index.js
  npm test
  ```

  ## Deploy

  1. Push this repo to GitHub
  2. Connect repo to Railway (New Project → Deploy from GitHub)
  3. Set `ANTHROPIC_API_KEY` and `PROXY_TOKEN` in Railway → Variables
  4. Railway auto-deploys on every push to main
  ```

- [ ] **Step 2: Commit README**

  ```bash
  git add README.md
  git commit -m "docs: add README with setup and deploy instructions"
  ```

- [ ] **Step 3: Verify `.env` is NOT tracked before pushing**

  ```bash
  git status
  ```

  Confirm `.env` does **not** appear in the output. It should be invisible because it's in `.gitignore`. If it shows up, stop and check your `.gitignore` before proceeding.

- [ ] **Step 4: Create GitHub repo and push**

  1. Go to github.com → New repository
  2. Name: `ocsc-proxy`
  3. Visibility: **Private** (keep API key setup instructions private)
  4. Do NOT initialize with README (you already have one)
  5. Copy the remote URL, then:

  ```bash
  git remote add origin https://github.com/<your-username>/ocsc-proxy.git
  git branch -M main
  git push -u origin main
  ```

---

## Chunk 8: Railway deployment

### Task 8: Deploy on Railway

> No code changes in this task — this is all in the Railway web UI and terminal.

- [ ] **Step 1: Create Railway account**

  Go to railway.app → Sign up with GitHub.

- [ ] **Step 2: Create new project**

  Railway dashboard → New Project → Deploy from GitHub repo → select `ocsc-proxy`.

  Railway will start deploying immediately. It will fail (missing env vars) — that's expected.

- [ ] **Step 3: Set environment variables**

  In Railway: your project → Variables tab → Add:
  - `ANTHROPIC_API_KEY` = your real Anthropic API key
  - `PROXY_TOKEN` = a random 32-character string

  To generate a random token (run in any terminal):
  ```bash
  node -e "console.log(require('crypto').randomBytes(16).toString('hex'))"
  ```

  Save the `PROXY_TOKEN` value — you'll need it in the frontend.

- [ ] **Step 4: Wait for auto-redeploy**

  Railway automatically deploys whenever you push to `main` — and also triggers a redeploy when you save environment variables. After adding the env vars, watch the deploy logs in the Railway dashboard. Look for:
  `ocsc-proxy listening on port <PORT>`

  > **Important:** Every future `git push origin main` will trigger a new production deploy automatically.

- [ ] **Step 5: Note your Railway URL**

  Railway → your service → Settings → find the public domain.
  It looks like: `ocsc-proxy-production-xxxx.up.railway.app`

  Save this URL.

- [ ] **Step 6: Smoke test the deployed proxy**

  ```bash
  curl https://<your-railway-url>/health
  ```
  Expected: `{"status":"ok"}`

  ```bash
  curl -X POST https://<your-railway-url>/api/claude \
    -H "Content-Type: application/json" \
    -H "X-Proxy-Token: <your PROXY_TOKEN>" \
    -d '{"model":"claude-haiku-4-5-20251001","max_tokens":10,"messages":[{"role":"user","content":"Say hi"}]}'
  ```
  Expected: real Anthropic response.

---

## Chunk 9: Wire frontend to proxy

### Task 9: Update `index.html` in `tsc-calc`

**Files:**
- Modify: `tsc-calc/index.html` (in the main repo, not this one)

> The proxy isn't used for any existing feature yet — this task adds the constants so they're ready when PDF parsing ships. The frontend doesn't call the proxy today; this just ensures the plumbing is in place.

- [ ] **Step 1: Add proxy constants to `index.html`**

  In `index.html`, find the opening `<script>` tag near the top of the JavaScript section. Add these two constants at the very top:

  ```js
  // Backend proxy — ocsc-proxy on Railway
  const PROXY_URL = 'https://<your-railway-url>';
  const PROXY_TOKEN = '<your PROXY_TOKEN value>';
  ```

  > **Security note:** `PROXY_TOKEN` in frontend JS is an accepted trade-off (documented in the design spec). Once committed to a public GitHub repo, treat this token as semi-public — anyone who looks at the source can see it. Its purpose is to stop automated abuse and bots, not to provide cryptographic security. Do not reuse this token for anything else. If abuse occurs, rotate it by changing the Railway env var and updating this constant.

- [ ] **Step 2: Commit to tsc-calc**

  ```bash
  git add index.html
  git commit -m "feat: add backend proxy URL and token constants"
  ```

- [ ] **Step 3: Verify the constants are correct**

  Open the app in a browser and open the browser console (F12 → Console tab). Paste:
  ```js
  console.log(PROXY_URL);
  console.log(PROXY_TOKEN);
  ```
  Confirm both values match what you set in Railway. Then verify all existing tabs (Builder, Label, Saved, Goals, Filter, Compare, About) still work — the constants are unused by existing code and should not affect anything.

---

## Final Checklist

- [ ] All 5 unit tests pass locally (`npm test` in `ocsc-proxy/`)
- [ ] Health endpoint live on Railway
- [ ] Real Anthropic call works through deployed proxy
- [ ] `PROXY_URL` and `PROXY_TOKEN` constants added to `index.html`
- [ ] No secrets committed to either repo
