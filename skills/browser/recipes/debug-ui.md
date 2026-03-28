# Recipe: Debug UI

## When to use
User is investigating a frontend bug — console errors, network failures, layout issues, unexpected behavior.

## Required capabilities
Exploratory interaction; optionally network inspection.

## Preferred tool -> Fallback
**native** (interactive, already authenticated) -> **agent-browser** (fast snapshots for exploration) -> **Playwright** (when network inspection or request interception is specifically needed)

Routing note: default to interactive/exploratory tools. Only reach for Playwright when the user specifically needs network-level debugging (request/response inspection, HAR recording).

## Prerequisites
- Page or dev server accessible (preflight: `curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT`)
- Know the URL and rough area of the bug

## Steps (native or agent-browser — exploratory)
1. Navigate to the problematic page:
   ```bash
   agent-browser open http://localhost:3000/buggy-page
   ```
2. Snapshot the page state:
   ```bash
   agent-browser snapshot -i
   ```
3. Look for visual issues in the accessibility tree (missing elements, wrong text, broken structure)
4. Screenshot for reference:
   ```bash
   agent-browser screenshot --annotated
   ```
5. If the bug is interactive, try to reproduce:
   ```bash
   agent-browser click @e1
   agent-browser snapshot -i
   ```
6. Report findings: what elements are present/missing, any visual anomalies, state after interaction

## Steps (Playwright — network debugging)
```javascript
import { chromium } from 'playwright';
const browser = await chromium.launch();
const context = await browser.newContext({
  recordHar: { path: '.browser-artifacts/debug.har' }
});
const page = await context.newPage();

// Capture console errors
page.on('console', msg => console.log(`[${msg.type()}] ${msg.text()}`));
page.on('pageerror', err => console.error('PAGE ERROR:', err.message));

// Capture network failures
page.on('requestfailed', req =>
  console.error(`FAILED: ${req.method()} ${req.url()} - ${req.failure().errorText}`)
);

await page.goto('http://localhost:3000/buggy-page');
// ... interact to reproduce the bug
await page.screenshot({ path: '.browser-artifacts/debug-screenshot.png' });
await context.close();
await browser.close();
```

## Output
- Console errors and warnings
- Failed network requests (if Playwright)
- DOM state summary (elements present, missing, unexpected)
- Annotated screenshot saved to `.browser-artifacts/`
- HAR file if network debugging was needed

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Page crashes | JS error or resource exhaustion | Capture last known state, check console for clues |
| Console too noisy | Many warnings/logs | Filter for `error` and `warning` types only |
| Can't reproduce | Bug is intermittent | Try multiple times, check for race conditions or timing |
| Auth required | Page behind login | Handle auth first (see SKILL.md auth modes) |

## Cleanup
Close browser session. HAR files and screenshots persist in `.browser-artifacts/`.
- agent-browser: `agent-browser close`
- Playwright: `await browser.close()`
