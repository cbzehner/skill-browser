# Recipe: Screenshot Diff

## When to use
User wants to visually compare a page across git branches, before/after a change, or across deploys.

## Required capabilities
Screenshot capture.

## Preferred tool -> Fallback
**agent-browser** (fastest for screenshots) -> Playwright -> native

## Prerequisites
- Git working tree is clean (`git status --porcelain` returns empty). If dirty, warn user and stop.
- Dev server running (or a way to start one) for the current branch
- Target URL is accessible

## Steps

### 1. Screenshot current branch
```bash
# Ensure .browser-artifacts/ exists
mkdir -p .browser-artifacts

# Screenshot with agent-browser
agent-browser open http://localhost:3000/target-page
agent-browser screenshot --full-page
# Save to .browser-artifacts/current-branch-name.png

# Or with Playwright (one-off script)
node -e "
import { chromium } from 'playwright';
const b = await chromium.launch();
const p = await b.newPage();
await p.goto('http://localhost:3000/target-page');
await p.screenshot({ path: '.browser-artifacts/current.png', fullPage: true });
await b.close();
"
```

### 2. Set up comparison branch via worktree
```bash
# Create a temporary worktree for the comparison branch
git worktree add /tmp/browser-diff-worktree main  # or whatever branch to compare

# Start a second dev server on a DIFFERENT port from the worktree
cd /tmp/browser-diff-worktree
# Detect start command from package.json, use a different port
PORT=3001 npm run dev &
DIFF_SERVER_PID=$!

# Wait for server to be ready
until curl -s -o /dev/null http://localhost:3001; do sleep 1; done
```

### 3. Screenshot comparison branch
```bash
agent-browser open http://localhost:3001/target-page
agent-browser screenshot --full-page
# Save to .browser-artifacts/comparison-branch-name.png
```

### 4. Present results
Show both screenshots to the user with clear labels:
- `current.png` -- [current branch name]
- `comparison.png` -- [comparison branch name]

Let the user visually compare. Note any obvious differences if visible from the screenshots.

### 5. Clean up (ALWAYS, even on failure)
```bash
# Kill the second dev server
kill $DIFF_SERVER_PID 2>/dev/null

# Remove the worktree
cd /original/project/dir
git worktree remove /tmp/browser-diff-worktree --force
```

## Output
- Two labeled screenshots in `.browser-artifacts/`
- Branch names clearly identified in filenames

## Failure modes
| Failure | Cause | Fix |
|---------|-------|-----|
| Dirty working tree | Uncommitted changes | Ask user to commit or stash first |
| Worktree creation fails | Branch doesn't exist | Check branch name, list available branches |
| Second server port conflict | Port 3001 already in use | Try ports 3002, 3003, etc. |
| Screenshot fails | Page not loading | Check server is actually running on expected port |
| Worktree removal fails | Files still in use | Kill server first, then force remove |

## Cleanup
**This cleanup MUST happen even on failure.** Use try/finally pattern:
1. Kill the comparison dev server process
2. Remove the git worktree
3. Close any open browser sessions

Never leave orphaned worktrees or server processes running.

## Gotchas
- **Viewport normalization**: Set a consistent viewport size for both screenshots so differences are real, not from different window sizes. Use `--viewport 1280x720` or equivalent.
- **Animation freezing**: If the page has animations, wait for `networkidle` and consider disabling CSS animations before screenshotting.
- **Dynamic content**: Timestamps, random content, or ads will create false diffs. Note this to the user.
