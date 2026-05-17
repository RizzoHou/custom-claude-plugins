# custom-claude-plugins

A personal marketplace of Claude Code plugins, opt-in per project.

## Plugins

- **`writing`** — academic writing toolkit (Chinese-first). Native Chinese drafting via DeepSeek, AI-trace scrubbing, CJK LaTeX/xeCJK build + visual audit, bibliography authenticity check, and a locked polish pipeline. Ships a PKU paper LaTeX template.
- **`claude-md-management`** — audit, score, and auto-improve `CLAUDE.md` files. Fork of the official Anthropic plugin; the `claude-md-improver` skill applies updates directly (no approval prompt). Revert via `git` if unwanted.

## Install

```bash
# Add this repo as a marketplace
/plugin marketplace add RizzoHou/custom-claude-plugins

# Install a plugin (does not enable it globally)
/plugin install writing@custom-claude-plugins
```

## Enable in a project

The plugins are intentionally **not enabled globally**. To turn one on in the current project, merge an `enabledPlugins` entry into the project's `.claude/settings.json`. For example, to enable `writing`:

```bash
mkdir -p .claude
jq '.enabledPlugins["writing@custom-claude-plugins"] = true' \
  .claude/settings.json 2>/dev/null \
  > .claude/settings.json.tmp \
  && mv .claude/settings.json.tmp .claude/settings.json \
  || echo '{"enabledPlugins":{"writing@custom-claude-plugins":true}}' \
       > .claude/settings.json
```

Then restart Claude Code in that project to load the plugin.

If you toggle plugins often, wrap the above in a local helper script and put it on `$PATH`. This repo's `.gitignore` reserves the pattern `plugins/*/[!.]*-init` for such per-plugin helpers (e.g. `plugins/writing/claude-writing-init`) — keep them out of the published tree since they typically encode personal paths.

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`.
2. Drop skills into `plugins/<name>/skills/<skill>/SKILL.md`.
3. Add an entry to `.claude-plugin/marketplace.json`.
