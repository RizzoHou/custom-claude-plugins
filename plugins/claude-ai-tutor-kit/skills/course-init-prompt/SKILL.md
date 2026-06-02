---
name: course-init-prompt
description: |
  Draft the Chinese init_prompt.md file that the user pastes into a fresh
  claude.ai session along with the per-section attachments. Pulls context
  from the section's textbook.pdf, the assignment list, any supplementary
  materials in the section folder (notes, handouts — noting scope mismatches
  on copied-whole notes), and (if present) the course's exam_analysis.md.
  When a prior-learning anchor file is attached (the user's own notes, tagged
  in config and copied with a `prior_` prefix), instructs the claude.ai
  session to connect the new material to concepts already in those notes.
  When a past_paper_problems file is attached, instructs the session to give
  direct, inline-quoted references to those past-exam problems (tagged by
  paper family) at the matching teaching moment, instead of vague "常考点".
  Then runs writing:humanizer-zh on the draft if the
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

You write the first message the user will paste into a new claude.ai session, alongside the section's attachments (textbook, assignment, and any supplementary materials). The prompt is in Chinese, ~30–40 lines, functional rather than literary.

## Inputs

A section folder, e.g. `splits/ch10/sec05/`, containing:

- `textbook.pdf` (already sliced)
- `assignment.md` (already transcribed)
- **0..N supplementary material files** — everything in the folder that is not `textbook.pdf`, `assignment.md`, or `init_prompt.md`. PDFs are copied whole, `.md` files are transcriptions; filenames signal origin (e.g. `official_notes_ch10.pdf`, `my_handwritten_notes.md`). A `prior_` prefix (e.g. `prior_notes_sec05.pdf`) marks a **prior-learning anchor** — the user's own record of what they have already learned, which the connect-to-prior instruction (see "Connecting to prior learning" below) keys off. Glob the folder for these and `Read` each to confirm what it is before describing it.
- **`past_paper_problems.md` / `past_paper_problems.pdf`** (optional) — statements of past-exam problems related to this section, each tagged by paper family (`直接 A 卷` / `同源旁证`), produced by `course-split-section` step 3b. When present, it drives the past-paper-reference instruction (see "Past-paper references" below).

Optionally, at the project root: `materials/exam_analysis.md` (produced by `course-exam-distill`). Use it if present.

## Output

`splits/chNN/secMM/init_prompt.md`, Chinese, ~30–40 lines.

## Required structure

1. **Title line**: e.g. `# 高数 A2 第十章第 5 节（幂级数）`.

2. **Attachment explanations** — one short paragraph per file actually present in the folder:
   - `textbook.pdf` — this section's body + 习题 X.Y, sliced from the scanned textbook.
   - `assignment.md` — this section's assigned problems (上交 / 不上交).
   - Each supplementary file — say what it is (read it to confirm: official chapter notes, your handwritten notes, a handout) and its form (whole copy vs transcription). If a copied-whole notes file's scope is broader than its name suggests (e.g. `official_notes_ch10.pdf` covers ch10 AND ch11), warn explicitly here. If there are no supplementary files, omit this and don't invent one.
   - `past_paper_problems.md` / `.pdf` (if present) — statements of related past-exam problems, tagged by paper family. Say it is the source for the direct past-paper references (item 5).

3. **Course context** — one short paragraph: this section's place in the chapter's overall framework, dependencies on prior sections, and (if `exam_analysis.md` exists) one sentence on how heavily this section is tested in past papers. Do not over-claim from a thin sample.

4. **User's goal** — explicit statement: master the section and complete the assignment in under 1 hour, simplest solutions, no over-elaboration.

5. **Preferred teaching style** — top-down, two phases:
   - Phase 1: one or two paragraphs of the section's main thread (骨架). If a prior-learning anchor file is attached (see "Connecting to prior learning"), close Phase 1 by hooking the skeleton onto the relevant concept(s) already in those notes — e.g. "本节的核心 X 建立在你笔记里已有的 Y 之上".
   - Phase 2: drill into details across multiple rounds, with one or two examples per round. When a detail depends on, extends, or contrasts with something in the attached notes, say so and point at the specific concept — not a generic "回顾一下旧知识". When `past_paper_problems` is attached, add the past-paper-reference requirement (see "Past-paper references"): for any round whose topic matches a listed problem, quote that problem inline and tag its family — not a vague "这是常考点".

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

1. Read it (locally — it is **not** an attachment).
2. Find the rows / sections relevant to this chapter+section.
3. In Phase 1 context paragraph, add one sentence of weight signal: "根据近 N 年试卷，本节常考 X 与 Y。"
4. **Do not** tell the session "详见 `exam_analysis.md`" — that file is never uploaded, so it is a dangling pointer the stateless session cannot follow. If you want it to look at concrete problems, point at the attached `past_paper_problems` file instead (see below). One sentence of weight signal here is enough; the actual problems are handled by the past-paper-reference instruction.

If `exam_analysis.md` does NOT exist, skip the weight-signal sentence rather than inventing one. Do not say "本节是重点" without evidence.

## Past-paper references

This is what turns a vague "本节是常考点" into the direct, quotable references the user asked for. **Only when `past_paper_problems.md` / `.pdf` is attached** in the section folder:

1. Add a short instruction (woven into the Phase 2 teaching style, item 5) telling the session: for each detail round, if the round's topic matches a problem in `past_paper_problems`, **quote that problem's statement inline** and state how it relates to what is being taught — rather than only asserting the topic is tested.
2. **Require a paper-family tag on every reference.** The file tags each problem `直接 A 卷` or `同源旁证`; the session must carry that through — e.g. "这正是 2024 春 A 期末第 6 题（直接 A 卷）考过的：……" vs "2023 春 B 期末第 4 题（B 卷，仅作旁证）有类似一题：……". The sample of direct-family papers is usually thin, so an unlabeled reference risks passing a sibling-family problem off as direct evidence — the tag prevents that.
3. **Grounded only.** Reference only problems actually in the attached file. If the session thinks a topic is tested but no matching problem is attached, it should say so plainly, not fabricate a paper/题号.

If no `past_paper_problems` file is attached, omit all of this — do not instruct the session to reference papers it cannot see.

## Connecting to prior learning

Attaching the user's own notes exists to anchor *new* material to what they have *already* learned. Make that explicit in the prompt — under two conditions:

1. **Only when a prior-learning anchor is attached.** `course-split-section` names such files with a `prior_` prefix, from a `supplementary_material` config entry the user tagged as a prior-learning anchor. If no `prior_`-prefixed file is in the folder, omit all connect-to-prior wording — do not invent an anchor, and do not treat the current section's official notes as prior learning.
2. **Grounded to the attached notes only.** The claude.ai session is stateless: it remembers nothing from earlier sessions and sees only this section's attachments. Instruct it to connect to "我上传的笔记里已有的概念", not to "我学过的一切" — telling it to connect to an unattached history invites confabulation about what the user knows. If a connection needs a concept not in the attached notes, the session should say so rather than assume it.

When the condition holds, weave the connection into the teaching style (item 5): Phase 1 hooks the skeleton onto a named prior concept, Phase 2 flags dependencies / extensions / contrasts per detail round. Do not bolt it on as a separate "现在复习旧知识" step.

## What you do NOT do

- Do not invent prerequisites you cannot verify from the textbook or notes.
- Do not tell the session to connect to prior knowledge that isn't in the attached `prior_` notes — grounded connections only, or it will confabulate the user's background.
- Do not point the session at `exam_analysis.md` (never uploaded) or instruct past-paper references when no `past_paper_problems` file is attached — same confabulation risk, applied to exam history.
- Do not over-promise ("一小时内一定能掌握" is fragile — the user already says "目标 1 小时", keep it as a goal, not a guarantee).
- Do not produce a 100-line prompt. 30–40 lines is the target.

## Report

Tell the user:

- Where you wrote the file.
- Whether `exam_analysis.md` was consulted, and whether a past-paper-reference instruction was added (how many problems the attached `past_paper_problems` file holds) or skipped (no file attached).
- Whether `humanizer-zh` ran or was skipped.
- A one-line summary of the section's main thread that you put in Phase 1, so the user can sanity-check before pasting into claude.ai.
