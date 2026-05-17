---
name: pdf-audit
description: After generating a PDF from source (LaTeX, Typst, Pandoc, browser print, etc.), visually audit the rendered output via the Read tool — never rely on source-level review alone. Use whenever the task is "generate / fix / improve a PDF" or any other artifact that Read ingests as an image (PPT/PPTX rendered to PDF, image transforms like crop/resize/recolor).
allowed-tools:
  - Read
  - Bash
metadata:
  trigger: After generating or modifying a PDF; after any image transform (crop, resize, recolor)
---

# pdf-audit

The PDF is the ground truth, not the source. Source-level review confirms intent; auditing the PDF confirms the *output*. The same applies to anything else `Read` ingests as an image.

## Why this matters

Source-level review repeatedly misses real rendering problems:

- Text overflow into margins
- Broken or missing figures
- Mis-sized fonts
- Garbled CJK glyphs (tofu)
- Overlapping text
- Blank pages
- Broken cross-references
- Mis-placed floats

Tools like ffmpeg's `crop` filter silently clamp out-of-bounds requests and produce dimension-correct garbage that only a visual check catches.

## How to apply

Treat the PDF as the ground truth whenever the task is "generate / fix / improve a PDF":

1. **Read 1–2 representative page ranges first** — typically the cover/title page plus one content page. This is a sanity check that the build succeeded and basic layout is right.
2. **Zoom into any page the user flagged or that the source diff touched.** If a section was edited, read that section's pages, not random pages.
3. **State what you actually saw on the page** — not just "PDF generated successfully." That wording is the giveaway that the audit was skipped.

Pass the PDF path to the `Read` tool; it ingests PDFs as images so you see the actual rendering.

## Cost discipline

Each page is an image and costs tokens. Don't dump every page on every change:

- Read the pages most likely to have changed.
- Plus one sanity-check page elsewhere.
- For a full audit (final delivery, suspected systemic issue), render every page — but state explicitly that's what you're doing.

## Beyond PDFs

The same rule applies to anything `Read` ingests as an image:

- **Slides**: render PPT/PPTX to PDF first, then audit.
- **Image transforms** (crop, resize, recolor): `Read` the output and describe what you see. Don't trust dimension-correct output from filters that silently clamp invalid parameters.

## When to invoke

- Any task that generates or modifies a PDF.
- Any image-transform task using ffmpeg, ImageMagick, or similar tools that silently accept out-of-bounds parameters.
- Before reporting a PDF/image task as done.
