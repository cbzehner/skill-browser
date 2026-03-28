---
name: browser
description: Unified browser automation. Detects installed tools (agent-browser, Playwright, native), routes by capability, handles auth and cleanup. Use when the user needs to interact with a web page, test a UI, take screenshots, scrape content, debug a frontend, or record a session.
argument-hint: "[url or task description]"
allowed-tools: Bash, Read, Glob, Grep
# Note: Write/Edit intentionally excluded - this skill executes and reports, it does not modify project files
---

# Browser

Detect available browser tools, route tasks by capability, handle auth and cleanup.

## Trigger

Activate when the user asks to:
- Navigate to or interact with a web page
- Test a UI or write E2E tests
- Take a screenshot or record a video
- Scrape or extract data from a site
- Debug a frontend (console errors, network, layout)
- Compare pages visually across branches

## Detection & Preflight

On the first browser task in a session, run these probes to determine what is available. Cache results for the session.

```bash
# agent-browser: binary AND Chrome installed
command -v agent-browser 2>/dev/null && agent-browser --version

# Playwright: CLI AND browser binaries
npx playwright --version 2>/dev/null
npx playwright install --dry-run 2>/dev/null

# Claude Code native: check if browser MCP tools exist in current tool list
```

**Do not trust `command -v` alone.** A CLI can be installed without its browser binaries. Run the version/dry-run checks above to confirm the tool is actually usable.

If a tool is installed mid-session, re-detect when the user says they have installed it.

### Preflight Checks (before every recipe)

- **Localhost target?** Verify the server is reachable:
  ```bash
  curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT
  ```
- **Output directory writable?** Ensure `.browser-artifacts/` exists and is writable.
- **Screenshot-diff?** Verify the git working tree is clean (no uncommitted changes).

## Capability-Driven Routing

Route by what the task **requires**, not by task label. Identify the needed capabilities from this table, then pick the tool that covers the most.

| Capability needed | Best tool | Why |
|---|---|---|
| Deterministic scripted flow | Playwright | Test runner, retries, assertions, parallel execution |
| Exploratory interaction | agent-browser / native | Fast snapshots, semantic refs, interactive |
| Token-efficient page reading | agent-browser | Accessibility tree is 82% smaller than raw HTML |
| Video recording | Playwright | Only tool with native video capture |
| Network inspection | Playwright | Request interception, HAR recording, console capture |
| Multi-browser coverage | Playwright | Chrome, Firefox, WebKit |
| Zero-setup, already authenticated | native | Uses user's existing browser session |
| Structured data extraction | agent-browser | Snapshot refs enable precise targeted extraction |
| Screenshot capture | agent-browser / Playwright | agent-browser for speed, Playwright for full-page/element options |

**Tie-breaking order:** native > agent-browser > Playwright (least overhead first).

**Fallback changes the workflow.** When falling back from one tool to another, tell the user what changes. For example, falling back from Playwright to agent-browser for E2E testing means losing the test runner and switching to manual agentic assertions. Do not present fallbacks as interchangeable.

## Auth Modes

Try these in order. Stop at the first one that works.

1. **Native browser session** -- Claude Code native browser uses the user's existing login state. Zero config.
2. **CDP connection** -- Connect to the user's running Chrome via `--cdp-url`. Both agent-browser and Playwright support this. Avoids profile locking.
3. **Dedicated automation profile** -- A separate browser profile (not the user's active one) with saved storage state. Avoids the profile-locking crash from importing an active Chrome instance.
4. **Cookie/storage import** -- Export cookies or localStorage from a logged-in session, import into the automation browser. Works but cookies expire.
5. **Manual login checkpoint** -- Launch a headed browser, ask the user to log in manually, save session state. Last resort but always works.

**Never supported:** Never ask for passwords. Never store credentials in plaintext. Never attempt to bypass MFA programmatically.

## Failure Policy

When a browser tool fails mid-task:

1. **Capture the error** -- screenshot if possible, console output, exit code.
2. **Clean up** -- kill spawned browser processes (`pkill -f "chrome.*--remote-debugging"` or equivalent). Close open sessions.
3. **Report clearly** -- show what failed, what was attempted, and what state things are in.
4. **Do not silently retry** -- diagnose first (timeout? anti-bot? auth expired? missing dependency?).
5. **Suggest alternatives** -- if the failure is tool-specific, recommend trying with another available tool.

**Timeouts:** Default 30s per navigation. For slow pages, use explicit waits (`wait --load networkidle` for agent-browser, `waitForLoadState` for Playwright) rather than increasing the global timeout.

## Security & Privacy

- **Artifact storage:** All screenshots, videos, HAR files go to `.browser-artifacts/` in the project root. Add it to `.gitignore` if not already present.
- **Credential redaction:** Never log, display, or store passwords, tokens, or session cookies in output. If a HAR file contains auth headers, warn the user before saving.
- **Authenticated scraping:** Do not silently scrape content behind authentication. Tell the user what you are about to access and why.
- **No external upload:** Never send screenshots, videos, or scraped content to external services without explicit user approval.

## No Tools Found

When detection finds nothing installed:

1. Report what was checked and that no browser tools were found.
2. Recommend based on context:
   - **Default:** agent-browser -- AI-optimized, token-efficient, fast.
     ```
     npm i -g agent-browser && agent-browser install
     ```
   - **Testing or video recording needed:** also recommend Playwright.
     ```
     npm i -D @playwright/test && npx playwright install
     ```
   - **Python-only environment:** mention browser-use as an alternative.
     ```
     uv add browser-use
     ```
3. **Wait for user approval before installing anything.**

## References

- Tool reference cards: [tools/agent-browser.md](tools/agent-browser.md), [tools/playwright.md](tools/playwright.md), [tools/native.md](tools/native.md)
- Recipes: [recipes/test-app.md](recipes/test-app.md), [recipes/screenshot-diff.md](recipes/screenshot-diff.md), [recipes/scrape-page.md](recipes/scrape-page.md), [recipes/debug-ui.md](recipes/debug-ui.md), [recipes/record-session.md](recipes/record-session.md)

Read tool cards and recipes on demand when executing a task. Do not load them all upfront.
