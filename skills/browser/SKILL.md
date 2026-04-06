---
name: browser
description: Unified browser automation. Detects installed tools (agent-browser, Playwright, native), routes by capability, handles auth and cleanup. Use when the user needs to interact with a web page, test a UI, take screenshots, scrape content, debug a frontend, or record a session.
argument-hint: "[url or task description]"
arguments:
  - url_or_task
license: MIT
allowed-tools: Bash Read Glob Grep
# Note: Write/Edit intentionally excluded - this skill executes and reports, it does not modify project files
---

# Browser

## Trigger

Activate when the user asks to: navigate/interact with a web page, test a UI, take screenshots or video, scrape data, debug a frontend, or visually compare pages across branches.

## Detection & Preflight

On the first browser task in a session, run these probes to determine what is available. Cache results for the session.

```bash
command -v agent-browser 2>/dev/null && agent-browser --version
npx playwright --version 2>/dev/null
npx playwright install --dry-run 2>/dev/null
# Native: check if host exposes browser MCP tools
```

<!-- WHY: CLI can exist without browser binaries — command -v alone gives false positives -->
**Do not trust `command -v` alone** — run version/dry-run checks to confirm usability. Re-detect if user installs tools mid-session.

### Preflight (before every recipe)

- **Localhost target?** `curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT`
- **Output dir?** Ensure `.browser-artifacts/` exists
- **Screenshot-diff?** Verify git working tree is clean

## Reconnaissance (mandatory by default)

<!-- WHY: Without this, model clicks elements without verifying page state -->
Before any DOM interaction: look before you touch.

1. **Wait for load** — use `networkidle` to let async content settle
2. **Snapshot** — screenshot or accessibility snapshot before first interaction, save to `.browser-artifacts/`
3. **Verify targets exist** — confirm elements are present before acting (agent-browser: check refs, Playwright: `expect(locator).toBeVisible()`)

**On failure:** capture page state (screenshot, console errors, URL), report expected vs. found. Do not retry blindly.

**Skip with** `--skip-recon` for simple non-interactive automation (e.g., single screenshot of a stable page).

## Capability-Driven Routing

Route by what the task **requires**, not by task label. Identify the needed capabilities from this table, then pick the tool that covers the most.

| Capability | Best tool |
|---|---|
| Deterministic scripted flow (tests, assertions) | Playwright |
| Exploratory interaction | agent-browser / native |
| Token-efficient page reading | agent-browser (accessibility tree, 82% smaller) |
| Video recording / network inspection / multi-browser | Playwright |
| Zero-setup, already authenticated | native |
| Structured data extraction | agent-browser |
| Screenshot capture | agent-browser (speed) / Playwright (full-page/element) |

**Tie-breaking:** native > agent-browser > Playwright (least overhead first). When falling back, tell the user what capabilities change — do not present tools as interchangeable.

## Auth Modes

<!-- WHY: Ordered fallback prevents trying complex approaches first or asking for passwords -->
Try in order, stop at first success:

1. **Native session** — user's existing login state, zero config
2. **CDP** (`--cdp-url`) — connect to running Chrome, avoids profile locking
3. **Dedicated profile** — separate automation profile with saved storage state
4. **Cookie import** — export/import cookies (expire over time)
5. **Manual checkpoint** — headed browser, user logs in, save state. Last resort.

**Never:** ask for passwords, store credentials in plaintext, bypass MFA programmatically.

## Failure Policy

<!-- WHY: Prevents silent retries and zombie browser processes -->
On failure: capture error (screenshot, console, exit code) → clean up browser processes → report clearly → diagnose before retrying (timeout? anti-bot? auth? dependency?). If tool-specific, suggest an alternative tool.

**Timeouts:** 30s default. Use explicit waits (`networkidle`) for slow pages rather than increasing global timeout.

## Security & Privacy

<!-- WHY: Browser-specific risks (HAR with auth headers, authenticated scraping) aren't covered by general security guidance -->
- All artifacts go to `.browser-artifacts/` — add to `.gitignore`
- Never log/store passwords, tokens, or session cookies. Warn before saving HAR files with auth headers.
- Do not silently scrape authenticated content — tell the user what you're accessing and why
- Never upload artifacts to external services without explicit approval

## No Tools Found

Report what was checked. Recommend based on context (wait for user approval before installing):
- **Default:** `npm i -g agent-browser && agent-browser install`
- **Testing/video:** also `npm i -D @playwright/test && npx playwright install`
- **Python-only:** `uv add browser-use`

## References

- Tool reference cards: [tools/agent-browser.md](tools/agent-browser.md), [tools/playwright.md](tools/playwright.md), [tools/native.md](tools/native.md)
- Recipes: [recipes/test-app.md](recipes/test-app.md), [recipes/screenshot-diff.md](recipes/screenshot-diff.md), [recipes/scrape-page.md](recipes/scrape-page.md), [recipes/debug-ui.md](recipes/debug-ui.md), [recipes/record-session.md](recipes/record-session.md)

Read tool cards and recipes on demand when executing a task. Do not load them all upfront.
