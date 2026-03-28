# Claude Code Native Browser

## Detection
Check the current session's tool list for browser-prefixed MCP tools (`browser_navigate`, `browser_screenshot`, etc.).

There is no CLI command to detect this. Availability depends on the user having the **Claude in Chrome** extension installed and enabled.

## Setup
1. Install the "Claude in Chrome" extension (beta, Chrome/Edge only)
2. Enable browser integration in Claude Code settings
3. No additional CLI tools or browser installs needed

## Core Workflow
The native browser uses Playwright under the hood, exposed through Claude Code's MCP integration. Commands are MCP tools, not CLI invocations.

Typical flow:
1. Navigate to URL via MCP tool
2. Read page content (DOM access, console logs)
3. Take screenshots
4. Interact with elements (click, fill, etc.)

## Auth
**Biggest advantage:** Uses the user's existing browser login state. If the user is logged into a site in Chrome, the native browser has access to those sessions.

No separate auth configuration needed. No cookie import, no CDP, no profiles.

## Output Format
- Console logs from the page
- Screenshots (PNG)
- DOM access / page content
- Network information (limited compared to direct Playwright)

## When to Use
- **Zero-setup tasks** -- no tools installed, but native browser is available
- **Authenticated pages** -- user is already logged in, no auth setup needed
- **Quick inspections** -- navigate, screenshot, read content
- **Simple interactions** -- click, fill, read results

## When to Defer Entirely
If native browser MCP tools are available and the task is simple (navigate + screenshot + read), use them directly **without invoking this skill's routing logic**. The overhead of detection and routing isn't worth it for one-shot tasks.

## Strengths
- Zero extra installs
- Already authenticated (user's browser session)
- Integrated with Claude Code's workflow
- Playwright-based under the hood

## Limitations
- Beta feature -- may change or have bugs
- Chrome/Edge only
- Requires the extension to be installed and enabled
- Less control than direct Playwright (subset of capabilities)
- No video recording
- Limited network inspection compared to direct Playwright
- Cannot run headless (uses the user's visible browser)

## Common Failures
| Failure | Cause | Fix |
|---------|-------|-----|
| MCP tools not available | Extension not installed or not enabled | Ask user to install/enable Claude in Chrome extension |
| Page not accessible | Browser not open or on different page | Ask user to open Chrome with the extension active |
| Auth not working | User not logged into the target site | Ask user to log in manually in Chrome first |
