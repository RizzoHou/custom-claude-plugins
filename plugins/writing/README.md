# writing

Academic writing toolkit for Claude Code, Chinese-first.

## What's in the box

### Skills

| Skill | Purpose |
|---|---|
| `consult-zh` | Route native Chinese drafting/critique to DeepSeek via the `consultant` CLI in a draft+critique flow. |
| `de-AI-writing` | Strip AI-template traces from Chinese prose while preserving the author's voice. |
| `humanizer-zh` | Detect and rewrite generic AI-writing patterns (破折号 over-use, 三段式, 否定式排比, etc.). |
| `latex-cjk-pdf` | Build, audit, and fix CJK PDFs via xelatex + xeCJK. Includes the SC+TC font hybrid for directional quote positioning. |
| `pdf-audit` | After generating any PDF (LaTeX/Typst/Pandoc/browser-print), visually audit the rendered output via `Read`. |
| `zh-prose-polish` | The locked polish pipeline: draft → `consult-zh` → `humanizer-zh` → `de-AI-writing`, with measurable targets. |
| `bib-verify` | Bibliography authenticity check — WebSearch every entry, flag fabrication hotspots (Chinese journals, edited volumes, secondary cites). |

### Templates

- **`templates/pku-paper/`** — Peking University paper template (xelatex + ctexart). Default for PKU-context papers unless the user specifies otherwise.

## Enable in a project

Merge the enable flag into the project's `.claude/settings.json`, then restart Claude Code in that project:

```bash
mkdir -p .claude
jq '.enabledPlugins["writing@custom-claude-plugins"] = true' \
  .claude/settings.json 2>/dev/null \
  > .claude/settings.json.tmp \
  && mv .claude/settings.json.tmp .claude/settings.json \
  || echo '{"enabledPlugins":{"writing@custom-claude-plugins":true}}' \
       > .claude/settings.json
```

See the marketplace README for guidance on wrapping this in a local helper script if you toggle it often.

## Prerequisites

- For `consult-zh`: the `consultant` CLI on `$PATH`, with a DeepSeek API key.
- For `latex-cjk-pdf` and the PKU template: a working TeX Live (xelatex + ctex + xeCJK), Noto Serif CJK fonts (both SC and TC for the hybrid quote trick), `poppler-utils` (`pdftoppm`, `pdftotext`).
