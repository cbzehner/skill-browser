# Recipe: Record Session

## When to use
User wants to capture a video or screenshot sequence of a user flow — for documentation, bug reports, or demos.

## Required capabilities
Video recording (Playwright only) or screenshot capture (any tool).

## Preferred tool -> Fallback
**Playwright** (native video recording) -> **agent-browser** (sequential screenshots — different artifact type, tell the user)

Tell the user upfront: Playwright produces a video file (WebM). agent-browser produces a numbered sequence of screenshots. These are different deliverables.

## Prerequisites
- Target flow is accessible
- `.browser-artifacts/` exists and is writable
- For video: Playwright available with browsers installed

## Steps (Playwright — video)
```javascript
import { chromium } from 'playwright';
const browser = await chromium.launch();
const context = await browser.newContext({
  recordVideo: {
    dir: '.browser-artifacts/videos/',
    size: { width: 1280, height: 720 }
  }
});
const page = await context.newPage();

// Walk through the flow
await page.goto('http://localhost:3000');
await page.waitForLoadState('networkidle');
await page.click('text=Sign In');
await page.fill('#email', 'user@example.com');
// ... more steps

// CRITICAL: close context to save video
await context.close();
await browser.close();
// Video is now at .browser-artifacts/videos/*.webm
```

## Steps (agent-browser — screenshot sequence)
```bash
mkdir -p .browser-artifacts/session-recording

agent-browser open http://localhost:3000
agent-browser screenshot
# Save as .browser-artifacts/session-recording/01-landing.png

agent-browser click @e1  # e.g. Sign In button
agent-browser snapshot -i
agent-browser screenshot
# Save as .browser-artifacts/session-recording/02-login-form.png

agent-browser fill @e2 "user@example.com"
agent-browser click @e3  # Submit
agent-browser snapshot -i
agent-browser screenshot
# Save as .browser-artifacts/session-recording/03-dashboard.png
```

Name files with numbered prefixes and descriptive suffixes so the sequence is clear.

## Output
- **Playwright:** Video file at `.browser-artifacts/videos/*.webm`. Report the file path.
- **agent-browser:** Numbered screenshots at `.browser-artifacts/session-recording/`. List all files with descriptions.

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Video not saved | Context not closed before browser | Always `await context.close()` before `browser.close()` |
| Video is black/empty | Page didn't render in time | Add `waitForLoadState('networkidle')` before interactions |
| Screenshot misses transient state | Action completes too fast | Add short waits or `waitForLoadState` between steps |
| Long flow exceeds timeout | Too many steps | Increase per-step timeout, or split into multiple recordings |

## Cleanup
Close browser session. Videos and screenshots persist in `.browser-artifacts/` for the user to review. Do not delete them automatically.
- agent-browser: `agent-browser close`
- Playwright: `await context.close()` then `await browser.close()`
