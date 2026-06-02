---
name: course-split-section
description: |
  Assemble a per-section study packet from a course's source materials.
  For one chapter+section (e.g. ch10 sec05), produces a folder containing:
  textbook.pdf (sliced to the section body + section exercises),
  assignment.md (transcribed from the aggregated assignment list),
  init_prompt.md (drafted next by the course-init-prompt skill, run as a
  separate step after), and 0..N supplementary materials (official or
  handwritten notes, handouts, anything else) each preserved by a
  fidelity-first rubric: copied whole when scanned, handwritten, or
  cross-referencing; transcribed when that helps Claude grasp it. When
  exam_analysis.md has a 原题索引, also pulls this section's related past-paper
  problems into past_paper_problems.md/.pdf (statements only, tagged by paper
  family) so the claude.ai session can quote them directly. Default layout is
  materials/ + splits/chNN/secMM/; alternate layouts via CLAUDE.md overrides
  or explicit user direction.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
metadata:
  trigger: split a course textbook section into a study packet folder
  scope: Chinese-textbook university courses; textbook is a scanned image-only PDF, supplementary materials may be any format
  default_materials_dir: materials
  default_splits_dir: splits
---

# course-split-section

You assemble a per-section study packet for one (chapter, section) of a Chinese-textbook university course.

## Inputs

The user names a chapter and section (e.g. "ch10 sec05") and may give a PDF page range. Default layout:

- `materials/textbook.pdf` — scanned image-only source.
- `materials/assignment_aggregation.pdf` — aggregated assignment list across all sections.
- **Supplementary materials** — 0..N items of any type: official chapter notes, your own handwritten notes, handouts, supplementary readings. Default location `materials/official_learning_notes/chNN_learning_notes.pdf` (back-compat); add more via CLAUDE.md config (below). These are the materials the preservation rubric in step 3 governs.
- `materials/exam_analysis.md` — optional, produced by `course-exam-distill`. Beyond the weight summary, its 原题索引 table (章/节 · 试卷 · 题号 · 源文件 · 页码 · 一行题意 · 卷别) is the lookup you use in step 3b to find past-paper problems related to this section.
- `splits/chNN/secMM/` — destination folder.

If the project uses a non-default layout, grep the project's root `CLAUDE.md` for a `## claude-ai-tutor-kit configuration` section. Inside, look for bulleted `- key: value` lines — relevant keys here are `materials_dir`, `splits_dir`, `assignment_aggregation_file`, and **every** `supplementary_material` line (this key repeats — collect all of them, not just the first). `learning_notes_pattern` is honored as a back-compat alias for a single official-notes source. Each `supplementary_material` value is a path or glob (supporting `{NN}` chapter / `{MM}` section tokens) with an optional parenthetical naming its type/scope. A parenthetical containing the literal phrase `prior learning anchor` flags the source as the user's own record of what they have already learned (e.g. `(my handwritten notes — prior learning anchor)`): name that source's destination with a `prior_` prefix (step 3) so `course-init-prompt` recognizes it as the notes to connect new material against. Match the literal phrase — do not infer anchor status from other wording. If the config section is absent and the default paths don't exist, ask the user before guessing — do not invent paths. If no supplementary materials are configured or found, proceed with textbook + assignment only and say so in your report.

## Output (per section)

In `splits/chNN/secMM/`:

1. `textbook.pdf` — sliced consecutive page range covering section body + section exercises (习题 X.Y). Always present.
2. `assignment.md` — transcribed assignment list for this section. Always present.
3. `init_prompt.md` — drafted by the `course-init-prompt` skill, called at the end. Always present.
4. **0..N supplementary material files** — one per supplementary source that applies to this section, each preserved per the rubric in step 3. Name each so its origin and form are obvious (e.g. `official_notes_ch10.pdf` for a verbatim copy, `my_handwritten_notes.md` for a transcription — the extension signals form: `.pdf` = copied as image, `.md` = transcribed text). A source tagged as a prior-learning anchor additionally takes a `prior_` prefix (e.g. `prior_notes_sec05.pdf`); that prefix is the contract `course-init-prompt` reads to know which notes to connect new material against. No separate manifest; the filenames carry the provenance.
5. **`past_paper_problems.md` (and/or `past_paper_problems.pdf`) — present only when past-paper problems map to this section** (step 3b). Holds the actual statements of related past-exam problems, each labeled by paper family, so the claude.ai session can quote them when teaching the matching topic. Omitted entirely when `exam_analysis.md` has no index or nothing maps to this section.

## Procedure

Everything you bring into the packet follows one principle: **preserve the information the section needs, in the form Claude can most precisely grasp in the claude.ai session.** Textbook and assignment are pre-decided instances of that principle — slice the relevant pages, transcribe only the problem numbers. Supplementary materials are open-ended, so step 3 gives you a rubric to reason with instead of a fixed answer.

### 1. Confirm or determine the page range

Image-only PDFs have no text layer; rely on visual inspection.

- If the user provided a PDF page range, use it.
- Otherwise, read candidate pages with `Read` (which ingests PDFs as images) until you locate the section's start page and end page. Section ends at the page after `习题 X.Y` finishes, or at the start page of the next section, whichever comes later.

**Boundary-page overlap rule:** if section M ends partway down a page where section M+1 begins, include that shared page in *both* sections' slices. Duplicating a few lines is harmless; missing content is not.

### 2. Slice the textbook

```bash
TMPDIR=$(mktemp -d)
pdfseparate -f START -l END <materials_dir>/textbook.pdf "$TMPDIR/page-%d.pdf"
pdfunite $(for i in $(seq START END); do echo -n "$TMPDIR/page-$i.pdf "; done) \
  <splits_dir>/chNN/secMM/textbook.pdf
rm -rf "$TMPDIR"
```

Substitute the actual `materials_dir` / `splits_dir` paths.

### 3. Bring in the supplementary materials

For each supplementary source that applies to this chapter/section, decide how to preserve it. The principle is **fidelity first, graspability second; when they conflict, fidelity wins.** What follows are priors, not laws — most materials fall into one of these cases, but if a material doesn't fit, reason from the principle rather than forcing a branch:

- **Copy whole, verbatim** (the common case) — for scanned/image-only PDFs, handwritten notes, anything that cross-references other sections, and anything you can't transcribe with high confidence. Claude reads images directly, so a faithful copy is usually both the most accurate and the most graspable form. Copy the *whole* unit (e.g. the full chapter file) rather than slicing when references span sections.
- **Transcribe to markdown** — when the source has a reliable text layer or is short and unambiguous, *and* a clean text form is easier to use than the raw scan. Transcription is a format change, not a summary: preserve all the information and visually verify against the source.
- **Excerpt / slice** — only when the material is cleanly separable by section *and* has no cross-section references. Most chapter notes fail this test.
- **Restructure or summarize** — only when the user explicitly opts in for that material. Never the default for primary reference material; it loses information.

**Cross-reference caution — the one that bites.** Before excerpting or slicing *any* material, check whether it references other sections or chapters (§4 cites §3, §2 cites §1). If it does, do not slice — copy the whole unit into every section folder under that chapter. This is why official chapter notes are copied whole, not per-section. If a file's name understates its scope (e.g. `ch10_learning_notes.pdf` actually covers ch10 AND ch11), record it in the project's CLAUDE.md so future runs do not re-discover the surprise.

Verbatim copy of a whole-chapter notes PDF (the default case):

```bash
cp <materials_dir>/official_learning_notes/chNN_learning_notes.pdf \
   <splits_dir>/chNN/secMM/official_notes_chNN.pdf
```

For other sources, substitute the configured path and a destination name that signals origin + form (`.pdf` for a copy, `.md` for a transcription). If the source's config parenthetical contains the literal phrase `prior learning anchor`, prefix the destination name with `prior_` (e.g. `prior_notes_sec05.pdf`).

### 3b. Pull in related past-paper problems

The claude.ai session is stateless and sees only what is uploaded. For it to give the user **direct, quotable references to past-exam problems** (the feature this step exists for), the actual problem statements must be in the packet — a pointer to `exam_analysis.md` does nothing, because that file is never uploaded.

1. **Gate.** Only run this step if `materials/exam_analysis.md` exists and contains an 原题索引 table. If it is missing or has no index, skip — produce no past-paper file and say so in the report. Do not invent problems.

2. **Look up this section's rows.** Read the 原题索引 and select rows whose 章/节 maps to this chapter/section. Match **generously and by topic, not just the label** — section mappings are heuristic and a problem may be filed under a *different* section than the one introducing the concept (e.g. a 对偶基 problem filed under 双线性函数 still belongs to the section that defines 对偶基). So also scan the 一行题意 column for this section's core concepts and pull those rows even when their 章/节 sits in another chapter. (`course-exam-distill` is told to emit a row per touched section, so the row should already exist — but match on topic anyway, in case an older index predates that rule.) **Cap at a handful** (≈3–6); when over the cap, keep `直接 A 卷` rows over `同源旁证`, and the rows whose 一行题意 is closest to this section's core. If you drop matches to stay under the cap, note it in the report — never silently truncate.

3. **Fetch each statement via its locator.** Use the row's 源文件 + 页码 to `Read` *only* the listed page(s) of the exam file. For a PDF, read just those page(s); for an image-file source (`.jpg/.png/…`, where 页码 reads `1`/`—`), `Read` the image directly — it ingests PDFs and images alike. Do not re-read the whole corpus — that is what the index spares you. Capture the **problem statement only**, not any solution; the session should teach the problem, not recite an answer.

4. **Preserve per the same fidelity rubric as step 3.** Clean text → transcribe into `past_paper_problems.md`. Matrices / diagrams / anything that garbles under transcription → `pdfseparate`/`pdfunite` the page(s) into `past_paper_problems.pdf` (the same garble caveat that applies to scanned exercises applies here). **If the source is an image file, not a PDF**, `pdfseparate`/`pdfunite` cannot process it (they are PDF-only tools and will error) — instead either transcribe the statement into `past_paper_problems.md`, or copy the image into the packet as `past_paper_problems_<paper>.<ext>` (e.g. `past_paper_problems_a2_2025.jpg`). Either form is fine; use both if some problems transcribe cleanly and others must be sliced or copied.

5. **Label every entry by paper family.** This labeling is the contract `course-init-prompt` relies on so the session never passes off a sibling-family problem as direct evidence. Tag each `直接 A 卷` or `同源旁证（B 卷 / 兄弟课程）`, and cite year/season/题号. Markdown shape:

   ```markdown
   # 与本节相关的历年真题

   ## 2024 春 A 期末 第6题  [直接 A 卷]

   <题目原文（仅题面，不含解答）>

   ## 2023 春 B 期末 第4题  [B 卷·旁证]

   <题目原文>
   ```

### 4. Transcribe the assignment

Open `materials/assignment_aggregation.pdf` and find this chapter+section's row. Write `splits/chNN/secMM/assignment.md` in Chinese:

```markdown
# X.Y <章节标题> 习题

## 上交部分

- 习题 1：(1)、(3)
- 习题 2：(5)、(7)

## 不上交部分

（本节无）
```

Only transcribe problem numbers — problem text lives in `textbook.pdf`. Use `（本节无）` when a section has nothing in a category.

**Known parsing quirk** (record in project CLAUDE.md if true for this course): some aggregated assignment PDFs have scrambled glyphs on the intro page but clean problem lists on subsequent pages.

### 5. Visually audit the outputs

Per the global `pdf.md` rule, read the rendered `splits/chNN/secMM/textbook.pdf` with the `Read` tool — check first page (does it begin where the section begins?) and last page (does it end at or after `习题 X.Y`?). Mention what you saw, not just "PDF generated successfully." If the boundary is off, re-slice.

Also check the supplementary materials you produced: for a verbatim copy, open the destination to confirm it is the right file and not empty/truncated; for a transcription, spot-check it against the source so no information was dropped or garbled. If you produced a `past_paper_problems.pdf`, open it to confirm the sliced pages are the right problems; if `.md`, spot-check each transcribed statement against the source page.

### 6. Hand off to `course-init-prompt`

Stop here. Do NOT invoke the init-prompt skill yourself — that skill is a separate step the user (or the parent agent) runs next. Decoupling means a failed init-prompt draft doesn't cost you the slicing work, and the init prompt can be re-drafted later without re-slicing the textbook.

In your final report (next step), tell the user explicitly: "Run `course-init-prompt` on `splits/chNN/secMM/` next."

### 7. Report

Tell the user:

- Which page range you used and why.
- One sentence on what you saw on the first and last pages of the sliced textbook.
- The destination folder.
- Each supplementary material you brought in, and how you preserved it (copied whole / transcribed / excerpted) in a few words — or, if none, that you proceeded with textbook + assignment only. Note which (if any) you named with a `prior_` prefix as a prior-learning anchor.
- Whether `exam_analysis.md` was consulted, and for the past-paper step (3b): how many related problems you brought in and by which family (`直接 A 卷` vs `同源旁证`), in what form (transcribed / sliced), any matches dropped to stay under the cap — or that the index was absent / nothing mapped, so no `past_paper_problems` file was produced.
- Anything you flagged for the project CLAUDE.md.
- That `course-init-prompt` is the next step.

## When to batch multiple sections

The user may ask for multiple sections in one go (e.g. "ch10 sec05 and sec06"). Process each fully — slicing, transcribing, init prompt — before moving to the next. Do not parallelize across sections; the boundary-overlap rule needs sequential awareness of where one ends and the next begins.

## What you do NOT do

- Do not edit anything under `materials/` — that's the instructor's source, treat as read-only.
- Do not generate placeholder or invented exercise text; if you can't read a page clearly, stop and tell the user.
- Do not summarize or drop supplementary material to save space; if you are unsure how to preserve it, default to a verbatim whole copy.
- Do not skip the visual audit — source-level correctness does not guarantee output-level correctness.
- Do not invent past-paper problems or guess their family. Only include problems that exist in the 原题索引 and whose statement you actually read from the source page; carry the index's family tag through. Include the problem statement only, never its solution.
