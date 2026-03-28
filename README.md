# Browser

Unified browser automation for Claude Code. Detects available tools, routes tasks by capability, and handles auth, failures, and cleanup.

> *One skill. Any browser tool. The right choice, automatically.*

## Why Use This?

- **Automatic tool detection**: Finds what's installed (agent-browser, Playwright, Claude Code native) and verifies it works
- **Capability-driven routing**: Picks the best tool based on what the task requires, not what you ask for
- **Auth handling**: Five modes from zero-config native sessions to manual login checkpoints
- **Clean failures**: Captures errors, kills browser processes, suggests alternatives

## Prerequisites

At least one of these browser automation tools:

| Tool | Install | Best For |
|------|---------|----------|
| **agent-browser** | `npm i -g agent-browser && agent-browser install` | Token-efficient reading, fast screenshots, structured extraction |
| **Playwright** | `npm i -D @playwright/test && npx playwright install` | E2E testing, video recording, network inspection, multi-browser |
| **Claude Code native** | Enable Claude in Chrome extension (beta) | Zero setup, already authenticated |

No tools required upfront. The skill detects what's available and recommends installs when needed.

## Installation

### From Marketplace

```bash
# Add the marketplace
/plugin marketplace add cbzehner/skill-browser

# Install the skill
/plugin install browser@cbzehner
```

### Manual Installation

Clone into your `.claude/skills/` directory:

```bash
cd ~/.claude/skills/
git clone https://github.com/cbzehner/skill-browser.git browser
```

## Usage

The skill activates automatically when you ask Claude to interact with a browser:

```
You: Take a screenshot of localhost:3000
You: Write E2E tests for the login flow
You: Scrape the pricing table from example.com
You: Debug the console errors on the dashboard page
You: Record a video of the signup flow
```

## Files

```
browser/
├── SKILL.md              # Main skill definition (detection, routing, auth, security)
├── tools/
│   ├── agent-browser.md  # Tool reference card
│   ├── playwright.md     # Tool reference card
│   └── native.md         # Tool reference card
├── recipes/
│   ├── test-app.md       # Write and run E2E tests
│   ├── screenshot-diff.md # Visual comparison across branches
│   ├── scrape-page.md    # Extract structured data from URLs
│   ├── debug-ui.md       # Inspect console errors, network, layout
│   └── record-session.md # Capture video/screenshots of a flow
├── docs/                 # Design docs
├── README.md             # This file
└── LICENSE               # MIT
```

## Security

- All artifacts (screenshots, videos, HAR files) go to `.browser-artifacts/`, gitignored
- Credentials are never logged or stored in plaintext
- Authenticated pages require explicit user acknowledgment
- Nothing is uploaded to external services without approval

## License

MIT
