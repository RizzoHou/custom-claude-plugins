# claude-ai-tutor-kit

Prepare per-section study packets for tutored learning sessions on [claude.ai](https://claude.ai). Scoped to **Chinese-textbook university courses** (scanned PDF textbook + a separate aggregated assignment list + chapter/section numbering). For other shapes of course material, you'll need to adapt the skills.

## Why this exists

The workflow assumes you are studying a course through chat-interface Claude sessions, one section at a time. For each section you upload three attachments (textbook excerpt, chapter learning notes, assignment list) and paste a Chinese init prompt that tells Claude what you want from this session. Preparing those four artifacts manually for every section is repetitive — these skills automate it.

## Skills

| Skill | When to run |
|---|---|
| `course-exam-distill` | **Once per course** (or after new exam papers are added). Reads everything under `materials/exam_papers/`, produces `materials/exam_analysis.md` with topic frequency, problem-type distribution, and a judgment call on high-leverage areas. |
| `course-split-section` | **Once per section.** Slices the source textbook PDF to cover section body + 习题 X.Y, copies the chapter learning notes verbatim (cross-references would break under slicing), transcribes the assignment list. Calls `course-init-prompt` at the end. |
| `course-init-prompt` | Invoked by `course-split-section`. Drafts the Chinese `init_prompt.md` and runs `writing:humanizer-zh` if the `writing` plugin is enabled in the project. |

## Default project layout

```
<course-project>/
├── materials/                              # instructor-provided, treat as read-only
│   ├── textbook.pdf
│   ├── assignment_aggregation.pdf
│   ├── official_learning_notes/
│   │   └── chNN_learning_notes.pdf
│   ├── exam_papers/                        # any nested layout under here
│   │   ├── spring_midterm/.../*.pdf
│   │   └── ...
│   └── exam_analysis.md                    # produced by course-exam-distill
└── splits/                                 # generated per section
    └── chNN/secMM/
        ├── textbook.pdf
        ├── official_learning_notes.pdf
        ├── assignment.md
        └── init_prompt.md
```

Alternate layouts are accepted. To override defaults, add a `## claude-ai-tutor-kit configuration` section to the project's `CLAUDE.md`:

```markdown
## claude-ai-tutor-kit configuration

- materials_dir: lectures
- splits_dir: notes
- exam_papers_dir: lectures/past_exams
- learning_notes_pattern: lectures/notes/ch{NN}.pdf
- assignment_aggregation_file: lectures/all_assignments.pdf
```

All keys are optional; missing keys fall back to the defaults shown above this snippet. Skills grep the project's `CLAUDE.md` for this section before guessing. If the section is absent and the default paths don't exist, skills ask the user rather than inventing locations.

## Prerequisites

- `pdfseparate` and `pdfunite` on `$PATH` (Poppler utilities — `apt install poppler-utils` or `brew install poppler`).
- Optional: the `writing` plugin from this same marketplace, for the `humanizer-zh` polish pass on init prompts.

## Enable in a course project

Merge the enable flag into the project's `.claude/settings.json` and restart Claude Code:

```bash
mkdir -p .claude
jq '.enabledPlugins["claude-ai-tutor-kit@custom-claude-plugins"] = true' \
  .claude/settings.json 2>/dev/null \
  > .claude/settings.json.tmp \
  && mv .claude/settings.json.tmp .claude/settings.json \
  || echo '{"enabledPlugins":{"claude-ai-tutor-kit@custom-claude-plugins":true}}' \
       > .claude/settings.json
```

If you toggle this often, see the marketplace README for the local-helper-script pattern.

## Typical first-time flow

1. Drop instructor materials under `materials/`.
2. Enable the plugin in `.claude/settings.json`, restart Claude Code.
3. Run `course-exam-distill` once. Sanity-check `materials/exam_analysis.md`.
4. For each section you want to study: `course-split-section` with the chapter and section identifiers. Review the generated `init_prompt.md`.
5. Upload the four files in the section folder to a fresh claude.ai session, paste the init prompt, study.

## Scope and limitations

- v0.1 is built around Chinese-textbook math/science courses. Courses with text-layer PDFs, embedded assignments, or non-Chinese materials may need the skills' assumptions adjusted.
- Exam-analysis judgment is heuristic — past papers are a signal, not a syllabus. Each year's paper can diverge from history.
- The `humanizer-zh` pass is a single editing pass tuned for functional Chinese prose. The full `writing:zh-prose-polish` pipeline is deliberately NOT invoked — it targets literary prose and over-edits instructions.
