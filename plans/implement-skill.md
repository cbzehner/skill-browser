---
status: complete
gaps: []
edge_cases: []
progress:
  - "Scaffold repo structure"
  - "Write SKILL.md"
  - "Write tool card: agent-browser"
  - "Write tool card: playwright"
  - "Write tool card: native"
  - "Write recipe: test-app"
  - "Write recipe: screenshot-diff"
  - "Write recipe: scrape-page"
  - "Write recipe: debug-ui"
  - "Write recipe: record-session"
  - "Symlink and verify"
last_review: 2026-03-27T22:08:00Z
iterations: 2
no_progress_count: 0
started_at: 2026-03-27T22:00:00Z
---

# Implement Browser Automation Skill

Spec: /Users/cbzehner/Developer/Personal/docs/superpowers/specs/2026-03-27-browser-skill-design.md

## Scaffold repo structure

Create the skill-browser repo at /Users/cbzehner/Developer/Personal/skill-browser/ following existing skill patterns (skill-magi, skill-ralph):
- Initialize git repo
- Create skills/browser/ directory with tools/ and recipes/ subdirectories
- Create LICENSE (MIT)
- Create README.md (brief, matching magi/ralph style)

## Write SKILL.md

Create skills/browser/SKILL.md with frontmatter, detection/preflight, capability-driven routing table, auth modes, failure policy, security section, and no-tools-found guidance. Should be the main entry point that references tool cards and recipes.

## Write tool card: agent-browser

Create skills/browser/tools/agent-browser.md — detection, install, core workflow (open/snapshot/interact/re-snapshot), output format (accessibility tree with refs), screenshots, auth, strengths, limitations, common failures.

## Write tool card: playwright

Create skills/browser/tools/playwright.md — detection with preflight (browser binaries), install, core workflow (test files, codegen), output formats (HTML/screenshots/video/traces), network inspection, AI agents (Planner/Generator/Healer), auth, strengths, limitations, common failures.

## Write tool card: native

Create skills/browser/tools/native.md — detection (check MCP tools in session), setup (Chrome extension), core workflow, output format, auth (existing browser session), strengths, limitations, when to defer entirely.

## Write recipe: test-app

Create skills/browser/recipes/test-app.md — E2E testing recipe. Required capabilities: deterministic scripted flow, assertions. Preferred: Playwright → agent-browser. Prerequisites, steps with actual commands, output, failure modes, cleanup.

## Write recipe: screenshot-diff

Create skills/browser/recipes/screenshot-diff.md — Visual comparison across branches. Uses git worktrees + second dev server on different port. Preferred: agent-browser → Playwright. Prerequisites (clean git state), steps, output, failure modes, cleanup (always remove worktree).

## Write recipe: scrape-page

Create skills/browser/recipes/scrape-page.md — Extract structured data from URLs. Preferred: agent-browser → native → Playwright. Steps for each tool, pagination handling, anti-bot failure mode, cleanup.

## Write recipe: debug-ui

Create skills/browser/recipes/debug-ui.md — Inspect console errors, network, layout. Preferred: native → agent-browser → Playwright. Exploratory interaction focus, steps, output format, failure modes.

## Write recipe: record-session

Create skills/browser/recipes/record-session.md — Capture video (Playwright) or screenshot sequence (agent-browser). Steps for each tool variant, artifact storage in .browser-artifacts/, cleanup.

## Symlink and verify

Create symlink from skills/browser to ~/.claude/skills/browser (replacing any existing link). Verify the skill is accessible.
