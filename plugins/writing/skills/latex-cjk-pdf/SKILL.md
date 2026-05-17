---
name: latex-cjk-pdf
description: |
  Build, audit, and fix CJK PDFs produced by xelatex + xeCJK. Use when generating
  or debugging Chinese-language LaTeX PDFs (reading reports, midterm essays,
  papers), or chasing CJK glyph dropouts (tofu), missing-character warnings, or
  quotation-mark direction bugs in the rendered output.
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
metadata:
  trigger: Building or debugging a CJK PDF via xelatex/xeCJK; visual audit after PDF rebuild
---

# latex-cjk-pdf

**The PDF is the ground truth, not the source.** Source-level review confirms intent; rendering confirms output. The bugs that bite a CJK xelatex pipeline — dropped glyphs, mis-routed fallback fonts, wrong-direction quotes — all pass source review and only show up in the rendered PDF.

After every rebuild, audit in two passes: log grep first (cheap, deterministic), then visual render of every page (catches what the log misses).

## 1. Audit after every rebuild (mandatory)

### 1.1 Grep the LaTeX log

```bash
grep -E "Missing character|Font shape.*undefined|There is no" build/<name>.log
```

Must come back empty. A `Missing character: There is no <X> in font [FandolSong-Regular.otf]` line is the *only* reliable signal that a CJK glyph dropped. **Visual inspection at low DPI can mistake a tofu/replacement glyph for a real Latin letter** — the canonical case is U+625E `扞` rendering as what looks like `F` because xetex emits a Latin-shaped substitute when the CJK code-point misses.

### 1.2 Render every page to PNG and Read each

```bash
pdftoppm -r 180 -png build/<name>.pdf /tmp/<name>-review/page
```

Then `Read` each `/tmp/<name>-review/page-*.png` in turn. **Don't skip pages.** Quote-direction bugs and CJK-fallback misroutes can hide on any page, and the log won't always flag them (a wrong quote glyph is still a valid glyph from the font's perspective).

State what you actually saw on the page, not just "PDF generated successfully" — that wording is the giveaway that the audit was skipped.

### 1.3 Cross-check with pdftotext (optional but cheap)

```bash
pdftotext build/<name>.pdf -
```

Then verify three properties on the extracted text:

- **Matched curly quotes**: `U+201C` count equals `U+201D` count. Unequal counts mean an opening or closing quote got dropped or substituted.
- **No stray ASCII straight quotes**: `U+0022` should not appear in CJK body text (see §3).
- **No CJK-tofu pattern**: `[一-鿿][A-Za-z][一-鿿]` — a Latin letter sandwiched between two CJK characters is almost always a tofu glyph, not a real character.

## 2. xeCJK fallback fonts: use `\newCJKfontfamily`, not `\newfontfamily`

To hand-route a single rare CJK code-point to a fallback font (e.g. when FandolSong is missing `扞` U+625E), declare the fallback with **`\newCJKfontfamily`** from xeCJK — *not* `\newfontfamily` from fontspec.

```latex
% WRONG: fontspec only swaps the Latin font. xeCJK keeps routing CJK
% code-points to the main CJK font, so the missing glyph stays missing
% and xetex emits a tofu that often looks like a Latin letter.
\newfontfamily\rarefallback{Noto Serif CJK SC}

% RIGHT: xeCJK knows to route this family for CJK code-points too.
\newCJKfontfamily\rarefallback{Noto Serif CJK SC}
```

The wrong form is the silent failure that produces a U+625E "扞 → F" bug: the source says `\rarefallback{扞}`, the log emits `Missing character: There is no 扞 in font [FandolSong-Regular.otf]`, and the PDF shows a Latin-shaped tofu where the CJK character should be.

## 3. Quotation marks in CJK body text

In source, prefer raw Unicode `"` (U+201C) and `"` (U+201D) directly over TeX-style `` `` `` `` and `''`. Both render correctly via xeCJK + FandolSong, but raw Unicode is unambiguous in source and survives copy-paste between `.md`, `.tex`, and the rendered PDF cleanly.

**Never use ASCII `"` (U+0022) in CJK body text.** FandolSong renders U+0022 as the right-double-quote glyph for *both* the opening and closing positions, producing the all-right-quote pairs that look correct at a glance but are wrong on the opening end. The fix is always to replace `"...."` with `"...."` at the source level.

## 4. SC + TC font hybrid: directional-quote routing

A given Unicode code-point — `U+201C/D` `""`, `U+2018/9` `''` — renders with subtly different glyph metrics depending on which CJK family routes it. In Noto Serif CJK, **SC** and **TC** place the directional quotes differently relative to adjacent CJK characters; one of the two will look right against the body text and the other will look misaligned (typically: floating too far from the character it should hug).

The fix is a hybrid: keep the body in the family you prefer for general text (commas, periods, body characters), and **route only the four directional quote code-points to the other family** if its directional-quote rendering looks better.

```latex
% Body family (whichever you prefer for general text)
\setCJKmainfont{Noto Serif CJK SC}

% Sub-family for directional quotes — use the other variant
\setCJKfamilyfont{quotesfont}{Noto Serif CJK TC}

% Apply via newunicodechar to the four directional quote code-points
\usepackage{newunicodechar}
\newunicodechar{“}{{\CJKfamily{quotesfont}\char"201C}}
\newunicodechar{”}{{\CJKfamily{quotesfont}\char"201D}}
\newunicodechar{‘}{{\CJKfamily{quotesfont}\char"2018}}
\newunicodechar{’}{{\CJKfamily{quotesfont}\char"2019}}
```

This is a surgical override: SC for everything except the four directional quote code-points, TC for those four. The grouping `{{\CJKfamily{quotesfont}…}}` keeps the family switch local so it doesn't leak into surrounding text.

If the body family preference is reversed (TC for body), flip `\setCJKmainfont` and `\setCJKfamilyfont` accordingly. The principle — route the four quote code-points to the family whose directional-quote rendering looks right — is symmetric.

Audit after applying: render at least one page containing quoted dialogue and verify the directional quotes hug the adjacent CJK characters as intended. If they still look misaligned, the override didn't apply — most often because `\newfontfamily` (from fontspec) was used instead of `\setCJKfamilyfont` / `\newCJKfontfamily` (from xeCJK), so the code-point still routes to the main CJK family. See §2.

## When to invoke

- Building any deliverable PDF whose body contains substantial CJK content via xelatex + xeCJK
- Debugging tofu glyphs, missing-character warnings, or Latin-letter substitutions in a rendered CJK PDF
- Auditing quotation-mark direction in a CJK manuscript before final submission

This skill is engine-specific to xelatex/xeCJK. lualatex + luatexja or pdflatex + CJK have similar but distinct pitfalls (different fallback APIs, different glyph-missing behavior); adapt the rules to the engine in use rather than copy them verbatim.
