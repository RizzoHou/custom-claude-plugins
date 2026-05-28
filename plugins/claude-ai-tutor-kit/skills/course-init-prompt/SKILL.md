---
name: course-init-prompt
description: |
  Draft the Chinese init_prompt.md file that the user pastes into a fresh
  claude.ai session along with the per-section attachments. Pulls context
  from the section's textbook.pdf, the chapter's learning notes (noting
  scope mismatches), the assignment list, and (if present) the course's
  exam_analysis.md. Then runs writing:humanizer-zh on the draft if the
  writing plugin is enabled. Functional instructions, not literary prose —
  do not run the full zh-prose-polish pipeline.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Skill
metadata:
  trigger: draft init_prompt.md for a course study section
  scope: Chinese-textbook university courses
---

# course-init-prompt

You write the first message the user will paste into a new claude.ai session, alongside three attachments. The prompt is in Chinese, ~30–40 lines, functional rather than literary.

## Inputs

A section folder, e.g. `splits/ch10/sec05/`, containing:

- `textbook.pdf` (already sliced)
- `official_learning_notes.pdf` (full chapter, may understate scope)
- `assignment.md` (already transcribed)

Optionally, at the project root: `materials/exam_analysis.md` (produced by `course-exam-distill`). Use it if present.

## Output

`splits/chNN/secMM/init_prompt.md`, Chinese, ~30–40 lines.

## Required structure

1. **Title line**: e.g. `# 高数 A2 第十章第 5 节（幂级数）`.

2. **Attachment explanations** — three short paragraphs:
   - `textbook.pdf` — this section's body + 习题 X.Y, sliced from the scanned textbook.
   - `official_learning_notes.pdf` — the chapter notes. If the file's scope is broader than its name suggests (e.g. `ch10_learning_notes.pdf` covers ch10 AND ch11), warn explicitly here.
   - `assignment.md` — this section's assigned problems (上交 / 不上交).

3. **Course context** — one short paragraph: this section's place in the chapter's overall framework, dependencies on prior sections, and (if `exam_analysis.md` exists) one sentence on how heavily this section is tested in past papers. Do not over-claim from a thin sample.

4. **User's goal** — explicit statement: master the section and complete the assignment in under 1 hour, simplest solutions, no over-elaboration.

5. **Preferred teaching style** — top-down, two phases:
   - Phase 1: one or two paragraphs of the section's main thread (骨架).
   - Phase 2: drill into details across multiple rounds, with one or two examples per round.

6. **Explicit ask before the assignment**: before tackling the assignment, the assistant should **map each `assignment.md` problem to its corresponding textbook exercise text** and tag the knowledge point each one tests. This catches mis-mapping early.

7. **Closing line**: `请从第 1 步开始。`

## Drafting style

- Functional, not flowery. Plain Chinese. No emojis.
- Avoid throat-clearing ("以下是这次会话的背景与我对你的期望"). Use direct openers ("下面是背景和我的期望" or just start with the title).
- Use numbered or bulleted lists where they aid scanning; do not bold every list item.
- Keep formatting consistent — if you bold the section heading, bold all section headings.

## Humanizer pass

After writing the draft, try to invoke the `writing` plugin's `humanizer-zh` skill via the `Skill` tool (`skill: "writing:humanizer-zh"`).

- **If the call succeeds**: apply the suggested edits surgically. Surgical means: keep changes within the original section structure, don't let the polish add throat-clearing or stylistic flourish that contradicts the "functional instructions" goal.
- **If the call errors (plugin not enabled in this project)**: leave the draft as-is, tell the user the `writing` plugin is not enabled, and continue. Do not block.

**Do not** run `writing:zh-prose-polish`. That pipeline targets literary/expository prose with metrics like "破折号 ≤ 6 per 5000 字". An init prompt is functional instructions and the pipeline will over-edit it.

## Exam-analysis integration

If `materials/exam_analysis.md` exists:

1. Read it.
2. Find the rows / sections relevant to this chapter+section.
3. In Phase 1 context paragraph, add one sentence like: "根据近 N 年试卷，本节常考 X 与 Y（详见 `exam_analysis.md`）。"
4. Do not list every past-paper problem — that's what `exam_analysis.md` is for. One sentence of weight signal is enough.

If `exam_analysis.md` does NOT exist, skip the weight-signal sentence rather than inventing one. Do not say "本节是重点" without evidence.

## What you do NOT do

- Do not invent prerequisites you cannot verify from the textbook or notes.
- Do not over-promise ("一小时内一定能掌握" is fragile — the user already says "目标 1 小时", keep it as a goal, not a guarantee).
- Do not produce a 100-line prompt. 30–40 lines is the target.

## Report

Tell the user:

- Where you wrote the file.
- Whether `exam_analysis.md` was consulted.
- Whether `humanizer-zh` ran or was skipped.
- A one-line summary of the section's main thread that you put in Phase 1, so the user can sanity-check before pasting into claude.ai.
