---
name: course-split-section
description: |
  Assemble a per-section study packet from a course's source materials.
  For one chapter+section (e.g. ch10 sec05), produces a folder containing:
  textbook.pdf (sliced from the source textbook to cover section body +
  section exercises), official_learning_notes.pdf (chapter notes copied
  verbatim because they cross-reference), assignment.md (transcribed from
  the aggregated assignment list), and init_prompt.md (drafted by the
  course-init-prompt skill, invoked at the end). Default layout is
  materials/ + splits/chNN/secMM/; alternate layouts are accepted via
  CLAUDE.md overrides or explicit user direction.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
metadata:
  trigger: split a course textbook section into a study packet folder
  scope: Chinese-textbook university courses with scanned image-only PDFs
  default_materials_dir: materials
  default_splits_dir: splits
---

# course-split-section

You assemble a per-section study packet for one (chapter, section) of a Chinese-textbook university course.

## Inputs

The user names a chapter and section (e.g. "ch10 sec05") and may give a PDF page range. Default layout:

- `materials/textbook.pdf` — scanned image-only source.
- `materials/official_learning_notes/chNN_learning_notes.pdf` — chapter notes.
- `materials/assignment_aggregation.pdf` — aggregated assignment list across all sections.
- `materials/exam_analysis.md` — optional, produced by `course-exam-distill`.
- `splits/chNN/secMM/` — destination folder.

If the project uses a non-default layout, grep the project's root `CLAUDE.md` for a `## claude-ai-tutor-kit configuration` section. Inside, look for bulleted `- key: value` lines — relevant keys here are `materials_dir`, `splits_dir`, `learning_notes_pattern`, `assignment_aggregation_file`. If the section is absent and the default paths don't exist, ask the user before guessing — do not invent paths.

## Output (per section)

Exactly four files in `splits/chNN/secMM/`:

1. `textbook.pdf` — sliced consecutive page range covering section body + section exercises (习题 X.Y).
2. `official_learning_notes.pdf` — full chapter notes, copied verbatim (see "verbatim rule" below).
3. `assignment.md` — transcribed assignment list for this section.
4. `init_prompt.md` — drafted by the `course-init-prompt` skill, called at the end.

## Procedure

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

### 3. Copy the chapter learning notes verbatim

```bash
cp <materials_dir>/official_learning_notes/chNN_learning_notes.pdf \
   <splits_dir>/chNN/secMM/official_learning_notes.pdf
```

**Verbatim rule, non-negotiable.** Chapter learning notes routinely have cross-section references (§4 cites §3, §2 cites §1). Slicing by textbook section breaks these references and produces a confused study companion. Always copy the full chapter file into every section folder under that chapter. If the file name understates its scope (e.g. `ch10_learning_notes.pdf` actually covers ch10 AND ch11), record this in the project's CLAUDE.md so future runs do not re-discover the surprise.

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

### 5. Visually audit the sliced textbook

Per the global `pdf.md` rule, read the rendered `splits/chNN/secMM/textbook.pdf` with the `Read` tool — check first page (does it begin where the section begins?) and last page (does it end at or after `习题 X.Y`?). Mention what you saw, not just "PDF generated successfully." If the boundary is off, re-slice.

### 6. Hand off to `course-init-prompt`

Stop here. Do NOT invoke the init-prompt skill yourself — that skill is a separate step the user (or the parent agent) runs next. Decoupling means a failed init-prompt draft doesn't cost you the slicing work, and the init prompt can be re-drafted later without re-slicing the textbook.

In your final report (next step), tell the user explicitly: "Run `course-init-prompt` on `splits/chNN/secMM/` next."

### 7. Report

Tell the user:

- Which page range you used and why.
- One sentence on what you saw on the first and last pages of the sliced textbook.
- The destination folder.
- Whether `exam_analysis.md` was consulted.
- Anything you flagged for the project CLAUDE.md.
- That `course-init-prompt` is the next step.

## When to batch multiple sections

The user may ask for multiple sections in one go (e.g. "ch10 sec05 and sec06"). Process each fully — slicing, transcribing, init prompt — before moving to the next. Do not parallelize across sections; the boundary-overlap rule needs sequential awareness of where one ends and the next begins.

## What you do NOT do

- Do not edit anything under `materials/` — that's the instructor's source, treat as read-only.
- Do not generate placeholder or invented exercise text; if you can't read a page clearly, stop and tell the user.
- Do not skip the visual audit — source-level correctness does not guarantee output-level correctness.
