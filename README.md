# custom-claude-plugins

A personal marketplace of Claude Code plugins, opt-in per project.

## Plugins

- **`writing`** — academic writing toolkit (Chinese-first). Native Chinese drafting via DeepSeek, AI-trace scrubbing, CJK LaTeX/xeCJK build + visual audit, bibliography authenticity check, and a locked polish pipeline. Ships a PKU paper LaTeX template.

## Install

```bash
# Add this repo as a marketplace
/plugin marketplace add RizzoHou/custom-claude-plugins

# Install a plugin (does not enable it globally)
/plugin install writing@custom-claude-plugins
```

## Enable in a project

The plugins are intentionally **not enabled globally**. To turn one on in the current project, run its `*-init` script. For example, to enable `writing`:

```bash
# from anywhere
/path/to/custom-claude-plugins/plugins/writing/writing-init
```

This merges `{"enabledPlugins": {"writing@custom-claude-plugins": true}}` into the project's `.claude/settings.json`. Restart Claude Code in that project to load the plugin.

Symlinking the init scripts onto `$PATH` keeps things ergonomic:

```bash
ln -s /path/to/custom-claude-plugins/plugins/writing/writing-init ~/.local/bin/writing-init
```

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`.
2. Drop skills into `plugins/<name>/skills/<skill>/SKILL.md`.
3. Drop a `<name>-init` script alongside (gitignored).
4. Add an entry to `.claude-plugin/marketplace.json`.
