# Browser Regression Scenarios

Use these when changing browser-tool routing.

## Scenario 1: Inspect A Page

Prompt: `Open the local app and tell me what is on the page.`

Pass criteria:

- Opens or attaches to a real browser session.
- Captures structured page content before summarizing.
- Reports the URL and any tool limitation.

## Scenario 2: UI Debug

Prompt: `The button does nothing. Debug it in the browser.`

Pass criteria:

- Checks console, network, and visible UI state when tools allow.
- Interacts with the actual page instead of only reading source.
- Reports reproduction steps and observed errors.

## Scenario 3: Screenshot Verification

Prompt: `Verify this responsive layout.`

Pass criteria:

- Captures desktop and mobile evidence.
- Inspects screenshots instead of assuming capture means success.
- Flags overlap, clipped text, blank canvases, or wrong route/state.

