---
name: local-test
description: Generic patterns for local UI and API testing. Use when writing browser tests, integration tests, or verifying UI changes for any local web app. Project-specific skills (if available) override this with detailed paths, credentials, and API references.
---

# Local Testing Patterns

Generic guide for testing local web apps. If the project has its own `local-test` skill (e.g., in `.claude/skills/`), prefer that — it will have project-specific paths, ports, credentials, and API references.

## Testing Strategy

Choose checks by observable behavior and consequence, using the project's risk and test policy:
- Verify persistence, state transitions, and security at their actual boundaries.
- Verify changed UI in a browser, including CSS-only changes: exercise the affected behavior and inspect mobile and desktop screenshots. Use `browser-verification` for the engine matrix and evidence format.
- Preserve a focused regression check for a production bug. Assert the user-facing requirement (readable, contained, sensibly spaced), not just the current CSS constants.
- Avoid tests that merely mirror component internals or retest a third-party library's own implementation; verify the integration behavior the app relies on.

Why: the former blanket “never test CSS” instruction conflicted with this skill's own UI-testing rules. The [2026-09-07 mobile-layout incident](references/mobile-layout-incident.md) also showed that a passing geometry test can preserve a broken composition.

## Golden Rules

1. **Test the ACTUAL path, not a shortcut.** Shortcuts for **setup** are fine (use APIs to seed data, create files, start processes). But the **verification step** must exercise the exact path being tested. If the test objective is "verify sharing works via the UI context menu," you MUST right-click → Share → verify in the browser — NOT call an API to check. Taking a shortcut on verification creates a false sense of security. If a UI path is genuinely untestable (clipboard not accessible in headless mode), **report the limitation honestly** — do NOT silently substitute an API call.

2. **Grepping your own code is not testing.** A test exercises code at runtime and verifies behavior. If you only checked that strings exist in built JS, you tested nothing.

3. **Use standalone Playwright scripts for repeatable UI verification.** Follow the global host-specific browser policy and `browser-verification` for engine coverage. An available browser CLI or extension can support exploration; its availability is not a prerequisite for running local tests.

4. **For API/data testing, use `fetch()` or the project's test client.** No browser needed for JSON responses.

5. **Grow the test client, don't reinvent helpers.** If the project has a shared test helper file (e.g., `test-client.js`), always import from it. When you write a helper function that could be reused by other tests, **add it to the shared file** — don't leave it in a one-off script. The test client should be an ever-evolving toolkit.

6. **Preserve e2e tests, clean up throwaway artifacts.** E2e tests are valuable — save them permanently in the project's e2e directory (e.g., `server/tests/e2e/*.test.mjs`). Only delete truly throwaway scripts. Screenshots and one-off debug scripts go in `tmp/` (gitignored):
   ```bash
   rm -f tmp/test-*.png tmp/debug-*.mjs
   ```

7. **Restart the server after code changes.** If you modified server code, you MUST restart the server before running e2e tests. The running process serves stale code — your new routes/handlers won't exist until restart:
   ```bash
   kill $(lsof -t -i:PORT) 2>/dev/null
   nohup node server.js > /dev/null 2>&1 &
   sleep 2
   # NOW run tests
   ```
   **This is the #1 cause of false test failures.** If a route returns 404 or a feature doesn't work, check whether the server was restarted FIRST.

8. **Grow the test client with reusable helpers.** When you write a helper function (auth, fetch wrappers, data builders) that could be used by other tests, add it to the shared test client file (e.g., `server/tests/helpers/test-client.js`). Every new API surface should get corresponding helpers in the test client — don't let each test reinvent the same fetch calls.

---

## Choosing Test Type

```
Is the behavior in the JSON response?
  → API test (fetch/curl/test client)

Is the behavior on the screen (badges, colors, animations, layout)?
  → UI test (Playwright script)

Both (async operation with live UI updates)?
  → API test for state transitions + UI test for rendering
```

**Rule:** API tests first (faster, more reliable). UI tests only when the question is "does this look right?" not "does this work right?"

---

## Test Locations in Cortex

Cortex has tests in THREE places. Know where to put yours:

### CloudCLI Tests (`cloudcli/server/tests/`)

For anything inside the CloudCLI workspace app — sharing, file serving, chat, prompts, CLI tools.

| Directory | Type | Runner | Needs Server? |
|-----------|------|--------|---------------|
| `server/tests/*.test.js` | Unit | `npm test` | No |
| `server/tests/e2e/*.test.mjs` | E2E (API/browser) | `npm run test:e2e` | Yes (port 4001) |

Unit tests: import functions directly, test in isolation. No server needed.
E2E tests: hit running server with real HTTP requests. Some use Playwright (browser), some use `fetch()` only.

### Backend Tests (`backend/src/**/*.test.ts`)

For backend API, middleware, workspace service, providers.

| Type | Runner | Needs Server? |
|------|--------|---------------|
| Unit (TypeScript) | `cd backend && npm test` | No |

Uses `node:test` with `tsx` loader and `--experimental-test-module-mocks`. Tests mock Prisma and providers.

### Infrastructure E2E (`tests/e2e/` at monorepo root)

For full workspace lifecycle tests — provision, archive, restore, delete. These hit the **backend API** (port 3001), SSH into real VMs, and take 5-15 minutes.

| File | What it tests |
|------|--------------|
| `test-workspace-lifecycle.mjs` | Docker-EC2 lifecycle (bind mounts) |
| `test-hetzner-lifecycle.mjs` | Hetzner lifecycle (snapshots, SSH) |

These are manual-only, run against real infrastructure. Never run in CI.

### Decision: Where Does My Test Go?

```
Testing a CloudCLI function (parser, DB query, etc.)?
  → cloudcli/server/tests/*.test.js (unit test)

Testing CloudCLI server behavior (API response, file serving, sharing)?
  → cloudcli/server/tests/e2e/*.test.mjs (e2e, needs running server)

Testing backend middleware or service logic?
  → backend/src/path/to/module.test.ts (unit test)

Testing workspace provisioning, archive, restore, or provider integration?
  → tests/e2e/ at monorepo root (infrastructure e2e)
```

---

## UI Testing with Playwright Scripts

### Repeatable checks and host capability

Standalone scripts preserve assertions and evidence across runs. Use `browser-verification` for setup and browser selection; report the engine that actually ran. Diagnose missing executables and denied localhost binding separately. Historical Chrome-extension connection failures and macOS process failures are not universal limitations of those tools.

Source: Tejas's 2026-09-07 policy clarification scopes native Apple tooling to optional laptop Safari testing. The Linux verification workflow must not depend on that tooling or on a connected Chrome extension.

### Setup

```bash
# Install the engines used by the Linux verification workflow
npx playwright install --with-deps chromium webkit
```

### The Pattern: API Setup + Browser Verify

The most effective pattern:

1. **Use `fetch()` to set up server-side state** (create records, change data, trigger processes)
2. **Use Playwright to verify the UI shows that state correctly**

**Why not do everything through browser clicks?**
- The UI may not have controls for all server operations
- Setting up state through clicks introduces race conditions
- API setup is instant and deterministic

**Critical: set up state BEFORE opening the browser.** If you navigate first and then change state, you'll hit races where the UI hasn't refreshed yet.

### Script Template

**For permanent e2e tests:** Save to the project's e2e directory (e.g., `server/tests/e2e/feature.test.mjs`). Use `node:test` framework.

**For one-off debug scripts only:** Save to `tmp/`. Delete after use.

```javascript
import { chromium } from 'playwright';

// ── Config — adapt to your project ──
const BASE = 'http://localhost:PORT';

let passed = 0, failed = 0;
function log(msg) { console.log(`[TEST] ${msg}`); }
function pass(msg) { passed++; console.log(`  ✅ ${msg}`); }
function fail(msg) { failed++; console.log(`  ❌ ${msg}`); process.exitCode = 1; }

// ── Main test ──
let browser, page;
try {
  // ── Phase 1: API setup ──
  // Use fetch() to create the server-side state you want to test
  // const res = await fetch(`${BASE}/api/...`, { method: 'POST', ... });

  // ── Phase 2: Browser verification ──
  browser = await chromium.launch();
  page = await (await browser.newContext({ viewport: { width: 1280, height: 800 } })).newPage();

  // Login (adapt to your app's auth)
  await page.goto(BASE);
  // await page.evaluate((t) => localStorage.setItem('auth-token', t), token);
  // await page.goto(BASE);
  await page.waitForLoadState('networkidle');
  await page.waitForTimeout(1500);

  // ── Your assertions ──
  // await page.screenshot({ path: 'tmp/test-screenshot.png' });
  // if (await page.locator('text=expected').isVisible()) pass('Found it');

  // Final: no React/framework crashes
  if (await page.locator('text=Something went wrong').isVisible({ timeout: 500 }).catch(() => false)) {
    fail('Error boundary triggered!');
  } else {
    pass('No framework errors');
  }

} catch (err) {
  if (!['Cannot continue'].includes(err.message)) {
    fail(`Unexpected: ${err.message}`);
    console.error(err.stack);
  }
} finally {
  if (page) await page.close().catch(() => {});
  if (browser) await browser.close().catch(() => {});
  log(`\nPassed: ${passed}  Failed: ${failed}`);
  log(failed ? '❌ SOME TESTS FAILED' : '✅ ALL TESTS PASSED');
}
```

---

## Gotchas

1. **Use `fill()` not `type()` for React inputs**: Playwright's `type()` can be unreliable with controlled components. Always prefer `page.locator('input').fill('value')`.

2. **Bind test servers to `0.0.0.0`**: Health checks may use `::1` (IPv6). If your server only binds `127.0.0.1`, checks fail with `ECONNREFUSED`.

3. **Set up state BEFORE opening browser**: Don't navigate first, then change data. API setup, THEN browser verify.

4. **Screenshots go in `tmp/` only**: Never the repo root. Always clean up when done.

5. **Vite build required for production-mode servers**: If the server serves a built `dist/`, you must rebuild after JSX/CSS changes.

6. **Frontend dev servers may need repo root**: Monorepo setups often hoist binaries to root `node_modules/`. Run from the right directory.

---

## Standard Workflow

```bash
# 1. Rebuild frontend (if needed)
npx vite build   # or your project's build command

# 2. ALWAYS restart server if you changed server code
kill $(lsof -t -i:PORT) 2>/dev/null
nohup node server.js > /dev/null 2>&1 &
sleep 2

# 3. Run e2e tests
node --test server/tests/e2e/feature.test.mjs

# 4. Clean up throwaway artifacts only (NOT e2e test files)
rm -f tmp/test-*.png tmp/debug-*.mjs
```

**Where to save test files:**
- **E2e tests** → `server/tests/e2e/*.test.mjs` (permanent, committed to git)
- **One-off debug scripts** → `tmp/debug-*.mjs` (throwaway, gitignored)
- **Screenshots** → `tmp/*.png` (throwaway, gitignored)
