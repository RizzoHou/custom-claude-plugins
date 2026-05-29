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
  cross-referencing; transcribed when that helps Claude grasp it. Default
  layout is materials/ + splits/chNN/secMM/; alternate layouts via CLAUDE.md
  overrides or explicit user direction.
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
- `materials/exam_analysis.md` — optional, produced by `course-exam-distill`.
- `splits/chNN/secMM/` — destination folder.

If the project uses a non-default layout, grep the project's root `CLAUDE.md` for a `## claude-ai-tutor-kit configuration` section. Inside, look for bulleted `- key: value` lines — relevant keys here are `materials_dir`, `splits_dir`, `assignment_aggregation_file`, and **every** `supplementary_material` line (this key repeats — collect all of them, not just the first). `learning_notes_pattern` is honored as a back-compat alias for a single official-notes source. Each `supplementary_material` value is a path or glob (supporting `{NN}` chapter / `{MM}` section tokens) with an optional parenthetical naming its type/scope. If the config section is absent and the default paths don't exist, ask the user before guessing — do not invent paths. If no supplementary materials are configured or found, proceed with textbook + assignment only and say so in your report.

## Output (per section)

In `splits/chNN/secMM/`:

1. `textbook.pdf` — sliced consecutive page range covering section body + section exercises (习题 X.Y). Always present.
2. `assignment.md` — transcribed assignment list for this section. Always present.
3. `init_prompt.md` — drafted by the `course-init-prompt` skill, called at the end. Always present.
4. **0..N supplementary material files** — one per supplementary source that applies to this section, each preserved per the rubric in step 3. Name each so its origin and form are obvious (e.g. `official_notes_ch10.pdf` for a verbatim copy, `my_handwritten_notes.md` for a transcription — the extension signals form: `.pdf` = copied as image, `.md` = transcribed text). No separate manifest; the filenames carry the provenance.

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

For other sources, substitute the configured path and a destination name that signals origin + form (`.pdf` for a copy, `.md` for a transcription).

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

Also check the supplementary materials you produced: for a verbatim copy, open the destination to confirm it is the right file and not empty/truncated; for a transcription, spot-check it against the source so no information was dropped or garbled.

### 6. Hand off to `course-init-prompt`

Stop here. Do NOT invoke the init-prompt skill yourself — that skill is a separate step the user (or the parent agent) runs next. Decoupling means a failed init-prompt draft doesn't cost you the slicing work, and the init prompt can be re-drafted later without re-slicing the textbook.

In your final report (next step), tell the user explicitly: "Run `course-init-prompt` on `splits/chNN/secMM/` next."

### 7. Report

Tell the user:

- Which page range you used and why.
- One sentence on what you saw on the first and last pages of the sliced textbook.
- The destination folder.
- Each supplementary material you brought in, and how you preserved it (copied whole / transcribed / excerpted) in a few words — or, if none, that you proceeded with textbook + assignment only.
- Whether `exam_analysis.md` was consulted.
- Anything you flagged for the project CLAUDE.md.
- That `course-init-prompt` is the next step.

## When to batch multiple sections

The user may ask for multiple sections in one go (e.g. "ch10 sec05 and sec06"). Process each fully — slicing, transcribing, init prompt — before moving to the next. Do not parallelize across sections; the boundary-overlap rule needs sequential awareness of where one ends and the next begins.

## What you do NOT do

- Do not edit anything under `materials/` — that's the instructor's source, treat as read-only.
- Do not generate placeholder or invented exercise text; if you can't read a page clearly, stop and tell the user.
- Do not summarize or drop supplementary material to save space; if you are unsure how to preserve it, default to a verbatim whole copy.
- Do not skip the visual audit — source-level correctness does not guarantee output-level correctness.
