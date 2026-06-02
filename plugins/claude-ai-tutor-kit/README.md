# claude-ai-tutor-kit

Prepare per-section study packets for tutored learning sessions on [claude.ai](https://claude.ai). Scoped to **Chinese-textbook university courses** (scanned PDF textbook + a separate aggregated assignment list + chapter/section numbering). For other shapes of course material, you'll need to adapt the skills.

## Why this exists

The workflow assumes you are studying a course through chat-interface Claude sessions, one section at a time. For each section you upload the section's attachments (textbook excerpt, assignment list, and any supplementary materials — official or handwritten notes, handouts, readings) and paste a Chinese init prompt that tells Claude what you want from this session. Preparing those artifacts manually for every section is repetitive — these skills automate it.

## Skills

| Skill | When to run |
|---|---|
| `course-exam-distill` | **Once per course** (or after new exam papers are added). Reads everything under `materials/exam_papers/`, produces `materials/exam_analysis.md` with topic frequency, problem-type distribution, a judgment call on high-leverage areas, and a per-problem **原题索引** (paper · 题号 · page locator · section mapping · paper-family tag) that `course-split-section` reads to route problems into packets. |
| `course-split-section` | **Once per section.** Slices the source textbook PDF to cover section body + 习题 X.Y, transcribes the assignment list, and preserves each supplementary material (notes, handouts) by a fidelity-first rubric — copied whole when scanned/handwritten/cross-referencing, transcribed when that aids grasp. When the 原题索引 exists, also pulls this section's related past-paper problems into `past_paper_problems.md`/`.pdf` (statements only, tagged by family). Hands off to `course-init-prompt` as the next step (you run it separately). |
| `course-init-prompt` | Run after `course-split-section` (a separate step, not auto-invoked). Drafts the Chinese `init_prompt.md`, has the claude.ai session connect new material to any attached prior-learning notes, give **direct inline-quoted references** to any attached past-paper problems (tagged by family) at the matching teaching moment, and runs `writing:humanizer-zh` if the `writing` plugin is enabled in the project. |

## Default project layout

```
<course-project>/
├── materials/                              # instructor-provided, treat as read-only
│   ├── textbook.pdf
│   ├── assignment_aggregation.pdf
│   ├── official_learning_notes/            # one kind of supplementary material;
│   │   └── chNN_learning_notes.pdf         #   handwritten_notes/, handouts/, … also fine
│   ├── exam_papers/                        # any nested layout under here
│   │   ├── spring_midterm/.../*.pdf
│   │   └── ...
│   └── exam_analysis.md                    # produced by course-exam-distill
└── splits/                                 # generated per section
    └── chNN/secMM/
        ├── textbook.pdf
        ├── assignment.md
        ├── init_prompt.md
        ├── official_notes_chNN.pdf         # 0..N supplementary files: .pdf = copied
        │                                   #   whole, .md = transcribed
        └── past_paper_problems.md          # optional: related past-exam problems
                                            #   (statements, tagged by paper family)
```

Alternate layouts are accepted. To override defaults, add a `## claude-ai-tutor-kit configuration` section to the project's `CLAUDE.md`:

```markdown
## claude-ai-tutor-kit configuration

- materials_dir: lectures
- splits_dir: notes
- exam_papers_dir: lectures/past_exams
- assignment_aggregation_file: lectures/all_assignments.pdf
- supplementary_material: lectures/official_notes/ch{NN}.pdf (official chapter notes, chapter-scoped)
- supplementary_material: lectures/my_notes/sec{MM}.pdf (my handwritten notes — prior learning anchor)
```

All keys are optional; missing keys fall back to the defaults shown above this snippet. `supplementary_material` is the one repeatable key — list it once per source, each value a path or glob with `{NN}` (chapter) / `{MM}` (section) tokens and an optional parenthetical naming its type/scope. Mark a source as a **prior-learning anchor** by putting `prior learning anchor` in its parenthetical — your own record of what you have *already* learned. `course-split-section` copies it with a `prior_` filename prefix, and `course-init-prompt` then has the claude.ai session actively connect each new concept to what is already in those notes (Phase 1 hooks the section's skeleton onto a named prior concept; Phase 2 flags dependencies and contrasts per detail round). Because the claude.ai session is stateless and sees only the files you attach, those connections are bounded to the attached notes — to link back to an earlier section, attach that section's notes too. Sections split before you added the tag won't have the `prior_` file; re-run `course-split-section` for them to pick up the anchor. `learning_notes_pattern` is still honored as a back-compat alias for a single official-notes source. Skills grep the project's `CLAUDE.md` for this section before guessing; if the section is absent and the default paths don't exist, they ask rather than inventing locations.

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
5. Upload `textbook.pdf`, `assignment.md`, and the supplementary files from the section folder to a fresh claude.ai session; paste `init_prompt.md`; study.

## Scope and limitations

- v0.1 is built around Chinese-textbook math/science courses. Courses with text-layer PDFs, embedded assignments, or non-Chinese materials may need the skills' assumptions adjusted.
- Exam-analysis judgment is heuristic — past papers are a signal, not a syllabus. Each year's paper can diverge from history.
- The `humanizer-zh` pass is a single editing pass tuned for functional Chinese prose. The full `writing:zh-prose-polish` pipeline is deliberately NOT invoked — it targets literary prose and over-edits instructions.
