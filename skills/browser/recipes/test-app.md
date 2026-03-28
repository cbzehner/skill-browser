# Recipe: Test App

## When to use
User asks to write, run, or fix E2E tests for a web application.

## Required capabilities
Deterministic scripted flow, assertions, optionally video recording.

## Preferred tool -> Fallback
**Playwright** -> agent-browser

Falling back to agent-browser means: no test runner, no built-in assertions, no parallel execution. You'll need to manually navigate, snapshot, and verify conditions via element inspection. Tell the user about this tradeoff.

## Prerequisites
- Dev server running and reachable (preflight: `curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT`)
- If Playwright: `npx playwright --version` succeeds and browsers installed
- If agent-browser: `command -v agent-browser` and Chrome available

## Steps (Playwright)
1. Check if `playwright.config.js` or `playwright.config.ts` exists
   - If not, create a minimal config:
     ```javascript
     import { defineConfig } from '@playwright/test';
     export default defineConfig({
       testDir: './tests',
       use: { baseURL: 'http://localhost:3000' },
     });
     ```
2. Write test file in `tests/` directory:
   ```javascript
   import { test, expect } from '@playwright/test';
   test('descriptive test name', async ({ page }) => {
     await page.goto('/');
     await expect(page.locator('h1')).toContainText('Expected');
     // ... interactions and assertions
   });
   ```
3. Run tests:
   ```bash
   npx playwright test
   ```
4. On failure, capture screenshot and report:
   ```bash
   npx playwright test --trace on
   npx playwright show-trace test-results/*/trace.zip
   ```
5. Report results: pass/fail count, failure details, screenshots of failures.

## Steps (agent-browser fallback)
1. Open the target page:
   ```bash
   agent-browser open http://localhost:3000
   ```
2. Snapshot to see the page state:
   ```bash
   agent-browser snapshot -i
   ```
3. Interact and verify manually:
   ```bash
   agent-browser click @e1
   agent-browser snapshot -i
   agent-browser get text @e2
   ```
4. Compare extracted text/state against expected values. Report pass/fail per check.
5. Screenshot on failure:
   ```bash
   agent-browser screenshot --annotated
   ```

## Output
- Test results summary (pass/fail per test)
- Screenshots of failures saved to `.browser-artifacts/`
- Trace files if Playwright with `--trace on`
- Video if Playwright configured with `video: 'retain-on-failure'`

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Dev server not reachable | Server not started | Ask user to start it, or detect the start command from package.json |
| Browser binary missing | Playwright installed but browsers not | `npx playwright install` |
| Test timeout | Selector not found or page too slow | Check selector, increase timeout, verify page loads |
| Flaky test | Race condition or animation timing | Add `await page.waitForLoadState('networkidle')` or use web-first assertions |

## Cleanup
- Playwright test runner handles browser cleanup automatically
- agent-browser: run `agent-browser close` after testing
- Video and trace files persist in `.browser-artifacts/` for user review
