# custom-claude-plugins

Personal marketplace of Claude Code plugins, opt-in per project. Public repo at github.com:RizzoHou/custom-claude-plugins.

## Layout

```
.claude-plugin/marketplace.json     # marketplace manifest, lists all plugins
plugins/<name>/
  .claude-plugin/plugin.json        # plugin manifest
  skills/<skill>/SKILL.md           # one dir per skill, frontmatter + body
  templates/<template>/             # optional bundled templates
  README.md
```

Marketplace `source` entries are relative: `"source": "./plugins/<name>"`.

## Conventions

- **Per-plugin init scripts are gitignored.** Pattern `plugins/*/[!.]*-init` in `.gitignore`. The script exists for local convenience (toggling `.claude/settings.json`); it is **not** a shipped artifact. Anything referenced from a committed README must itself be committed — do not write `/path/to/.../writing-init` style instructions for gitignored helpers.
- **Plugins ship disabled.** Enabling is per-project via `.claude/settings.json`: `{"enabledPlugins": {"<plugin>@custom-claude-plugins": true}}`. Never add to global `~/.claude/settings.json`.
- **Templates: PII gets `\newcommand` placeholders, not deletion.** When bundling a real-world LaTeX template, replace identifying fields with `\newcommand{\studentName}{...}` style placeholders so users fill in once and the value propagates.
- **Bundled third-party content (reference files, samples, style anchors) requires explicit redistribution review before pushing**, since this repo is public. The `writing` plugin's `de-AI-writing/references/{writing-samples,style-dna}.md` ship quoted prose from published essays — that decision was made consciously this session, not by default. Future skills bundling third-party content: ask the user before publishing.
- **Promoting a global skill into a plugin means deleting the global copy.** Global skills live in `~/projects/claude-code-config/skills/` (symlinked to `~/.claude/skills/`) and load unconditionally in every project — leaving a copy there preempts the plugin and defeats per-project opt-in. After packaging a skill under `plugins/<name>/skills/`, remove the matching directory from the config repo and commit.
- **Forking an upstream plugin is fine but document the delta.** When copying a plugin from another marketplace (e.g. `claude-plugins-official`) into `plugins/<name>/`: keep the original `LICENSE`, attribute the upstream source in the plugin's README, and call out in one sentence what the fork changes (default behavior, prompts, allowed tools). Mention the fork + delta in this repo's top-level `README.md` plugin list too. Once the fork is installed via `/plugin install <name>@custom-claude-plugins` and enabled in a project, the upstream copy can be removed via `/plugin uninstall <name>@<original-marketplace>`. Don't silently diverge — future sessions need to know whether to rebase from upstream or treat the fork as source of truth.

## Adding a new plugin

1. `plugins/<name>/.claude-plugin/plugin.json` — manifest.
2. `plugins/<name>/skills/<skill>/SKILL.md` — one or more skills.
3. Append an entry to `.claude-plugin/marketplace.json`.
4. Optional: local `plugins/<name>/<name>-init` script (gitignored).

## Updating an existing plugin

`claude plugin update` is **version-gated, not content-gated**: editing a `SKILL.md` without bumping the plugin's `version` makes update a silent no-op. To ship a change:

1. Edit, then **bump `version`** in `plugins/<name>/.claude-plugin/plugin.json` (new feature → minor bump).
2. Commit + **push to `main`** — the installed marketplace `source` is the GitHub repo, so update pulls from the remote, not the local working copy. Unpushed edits won't propagate.
3. `claude plugin marketplace update custom-claude-plugins`, then `claude plugin update <name>@custom-claude-plugins` (the `@marketplace` suffix is **required** — the bare name errors `not found`). Restart to apply; the new version installs to a fresh `cache/.../<version>/` dir.

## Testing a plugin loads

`/plugin marketplace add RizzoHou/custom-claude-plugins`, then `/plugin install <name>@custom-claude-plugins`, then enable in a test project's `.claude/settings.json` and restart Claude Code in that project.
