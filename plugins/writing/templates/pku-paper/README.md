# pku-paper

PKU academic paper template (xelatex + ctexart).

This is the default template for PKU-context papers when no other template is specified by the user.

## Files

- `main.tex` — the template entry point. Author identity is parameterized via `\newcommand` at the top; fill those in and the rest propagates to title page and page header.
- `pku-logo.png` — PKU校徽, embedded at 15% text width on the cover.
- `figures/` — drop figure files here; the example `\includegraphics` paths assume this location.

## Compile

```bash
xelatex main.tex
xelatex main.tex   # second pass populates \tableofcontents
```

Requires a working TeX Live with `ctex` + `xeCJK`. For better Chinese typography (mainland-style comma/period positioning with TC-style directional quotes), see the `latex-cjk-pdf` skill's SC+TC font hybrid section.

## What to fill in

At the top of `main.tex`:

```latex
\newcommand{\paperTitle}{在此填写论文题目}
\newcommand{\studentName}{在此填写姓名}
\newcommand{\studentID}{在此填写学号}
\newcommand{\studentDept}{在此填写院系}
\newcommand{\paperDate}{在此填写日期}
```

Then replace the placeholder body sections (引言, 文献综述, etc.) with actual content. The body sections are kept short on purpose — they're scaffolding, not lorem ipsum.

## Optional packages

- `pythonhighlight` is commented out by default. Uncomment to enable Python code highlighting (requires `pygments` installed locally and `--shell-escape` at compile time).

## Known limitations

- `subfigure` is deprecated upstream in favor of `subcaption`. It still works; replace if you hit issues.
- Bibliography uses the inline `thebibliography` environment, not BibTeX. For larger papers, swap to `\bibliography{refs.bib}` with `bibtex` or `biber`.
