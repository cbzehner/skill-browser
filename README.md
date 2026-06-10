> **Moved:** this skill now lives in [cbzehner/skills](https://github.com/cbzehner/skills) under `skills/browser/`. This repo is archived and read-only.

# Browser

Route browser work to the right local automation path. Use it for opening pages, inspecting UI, scraping, screenshots, console checks, and frontend verification.

## Use It For

- Checking a local app in a real browser
- Capturing screenshots or page text
- Debugging frontend console, network, or layout behavior

## Install

Clone the repo and run the installer:

```bash
git clone https://github.com/cbzehner/skill-browser.git
cd skill-browser
./install.sh all
```

Install targets:

- `./install.sh claude` installs to `~/.claude/skills/browser`
- `./install.sh codex` installs to `~/.codex/skills/browser`
- `./install.sh agents` installs to `~/.agents/skills/browser`
- `./install.sh opencode` installs to `~/.config/opencode/skills/browser`
- `./install.sh all --copy` copies files instead of symlinking

Manual install works too: symlink or copy `skills/browser` into your agent's skills directory.

## Agent Support

This repo uses the plain `skills/browser/SKILL.md` layout. Claude Code and Codex also get small plugin manifests at `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`.

Other agents can read the same `SKILL.md` file. If a host does not support a frontmatter field or tool name, ignore that field and follow the workflow text.

## Layout

```text
.claude-plugin/plugin.json
.codex-plugin/plugin.json
install.sh
skills/browser/SKILL.md
README.md
LICENSE
```

## Public Notes

These repos are public. Keep private repo names, secrets, customer data, raw logs, cookies, and absolute filesystem paths out of examples.

## License

MIT