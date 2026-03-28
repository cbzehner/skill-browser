# agent-browser

## Detection
```bash
command -v agent-browser 2>/dev/null && agent-browser --version
```
If command exists but version fails, Chrome may not be installed. Run `agent-browser install`.

## Install
```bash
npm i -g agent-browser && agent-browser install
```
Also available via `brew install agent-browser` or `cargo install agent-browser`.

## Core Workflow
The fundamental pattern is: navigate -> snapshot -> interact -> re-snapshot.

```bash
# 1. Open a page
agent-browser open https://example.com

# 2. Snapshot to see interactive elements with refs
agent-browser snapshot -i
# Returns accessibility tree with @e1, @e2, etc.

# 3. Interact using refs
agent-browser click @e1
agent-browser fill @e2 "text to type"
agent-browser select @e3 "option value"

# 4. Re-snapshot after any DOM change
agent-browser snapshot -i
```

**Critical: Refs are invalidated when the page changes.** Always re-snapshot after clicking links, submitting forms, or triggering dynamic content.

## Element Selection
- **Refs** (`@e1`) -- from snapshots, recommended for AI use
- **Semantic locators** -- `role=button[name="Submit"]`, `text=Click me`
- **CSS selectors** -- `#id`, `.class`, `div > span`
- **XPath** -- `//div[@class='content']`

Prefer refs. Fall back to semantic locators. Use CSS/XPath only when refs and semantics don't work.

## Screenshots
```bash
agent-browser screenshot                    # Standard
agent-browser screenshot --annotated        # With element labels
agent-browser screenshot --full-page        # Entire scrollable page
```
Save to .browser-artifacts/ directory.

## Data Extraction
```bash
agent-browser get text @e1          # Text content of element
agent-browser get url               # Current page URL
agent-browser get title             # Page title
```

## Batch Operations
Execute multiple commands in one invocation via JSON piping:
```bash
echo '[{"command":"open","args":["https://example.com"]},{"command":"snapshot","args":["-i"]}]' | agent-browser
```

## Auth
- **Import from browser:** `agent-browser auth import` (imports from user's Chrome)
- **CDP connection:** `agent-browser --cdp-url ws://localhost:9222`
- **Profile persistence:** Sessions persist across commands by default

## Output Format
Returns an **accessibility tree** with element references -- 82% less context than raw HTML. Example:
```
[page] Example Site
  [heading] Welcome
  [button @e1] Sign In
  [textbox @e2] Email
  [textbox @e3] Password
  [link @e4] Forgot password?
```

## Strengths
- Token-efficient (82% less context than Playwright/raw HTML)
- Fast (Rust native binary, no Node.js startup)
- AI-optimized element references
- Semantic understanding of page structure

## Limitations
- Chrome-only (no Firefox, Safari, WebKit)
- No test runner or assertion framework
- No video recording (use sequential screenshots as workaround)
- No network inspection or request interception
- Newer project, less documentation than Playwright
- Cannot run in parallel with another agent-browser session on same Chrome instance

## Common Failures
| Failure | Cause | Fix |
|---------|-------|-----|
| "Chrome not found" | `agent-browser install` was skipped | Run `agent-browser install` |
| Stale refs after click | Page changed, refs invalidated | Always re-snapshot after interactions |
| Timeout on slow page | Default 25s timeout | Set `AGENT_BROWSER_DEFAULT_TIMEOUT` env var or use explicit waits |
| Profile lock | Trying to use active Chrome profile | Use `--cdp-url` to connect to running Chrome instead |
