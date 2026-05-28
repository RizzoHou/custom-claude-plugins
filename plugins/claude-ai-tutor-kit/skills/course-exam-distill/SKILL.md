---
name: course-exam-distill
description: |
  Read all past exam papers under a course project's exam-papers directory
  (default `materials/exam_papers/`) and produce a single distilled analysis
  file (default `materials/exam_analysis.md`). The analysis tallies which
  chapters/sections/topics recur across years, common problem types per
  topic, and Claude's judgment of high-leverage areas to emphasize during
  study. Run this ONCE per course (or whenever new papers are added);
  per-section skills then read the analysis instead of re-reading every PDF.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
metadata:
  trigger: distill past-exam papers into a course-level study weight analysis
  scope: Chinese-textbook university courses with scanned image-only exam PDFs
---

# course-exam-distill

You distill a folder of past exam papers into a single study-weight analysis. Per-section study-prep skills read your output; you only run once per course.

## What you receive

A course project directory. Default layout:

- `materials/exam_papers/<category>/<paper>/<file>.pdf` — typically organized by `spring_midterm/`, `spring_final/`, `fall_midterm/`, `fall_final/`. Each paper folder may contain the exam PDF and student-authored solution PDFs.
- `materials/textbook.pdf` — for chapter/section reference.

If those default paths do not exist, grep the project's root `CLAUDE.md` for a `## claude-ai-tutor-kit configuration` section. Inside, look for bulleted `- key: value` lines — the relevant keys here are `materials_dir` and `exam_papers_dir`. If the section is absent and the default paths don't exist, ask the user before guessing.

## What you produce

`materials/exam_analysis.md` (or `<materials_dir>/exam_analysis.md`). Single file. Overwrite freely on re-runs.

## Procedure

1. **Inventory.** List every paper PDF under `exam_papers/`. Distinguish exam papers from student-authored solutions (filenames often contain `solution`, `answer`, `by_<name>`). Solutions are useful for cross-checking topic coverage but should not be double-counted.

2. **Identify the course code family.** If the current course is `advanced_mathematics_a2`, papers from sibling families (e.g. `advanced_mathematics_b2`) often share most topics but may have different difficulty / topic weighting. Note this in the analysis — do not assume `b2` papers are 1:1 representative of `a2` exams.

3. **Read each paper.** Most will be image-only scanned PDFs — use `Read` on the PDF path (it ingests pages as images). Do not skip reading because there is no text layer. For papers 10+ pages, read all pages; for shorter ones, the same.

4. **Per paper, log:**
   - Year, season, midterm/final, course code (e.g. `a2-2024-final`).
   - Problem types observed (filling blanks, computation, proof, applied).
   - Topics tested per problem, mapped to the textbook's chapter / section structure where possible. If unsure of mapping, write the topic in plain words and flag for review.

5. **Aggregate.** Build a topic frequency table: rows = topics (or chapter/section labels), columns = years/papers, cells = how many problems hit that topic. Also tally problem-type distribution per chapter.

6. **Make a judgment call.** Past papers are a signal, not a rulebook — each year's paper can diverge. Your analysis must include:
   - **High-leverage topics**: appear 3+ years out of 5, often worth 15%+ of the paper.
   - **Likely-tested but easy-to-miss**: appear less often but are computationally heavy or proof-style, so they merit careful prep.
   - **Possibly-tested edge topics**: appear once in the sample.
   - **Caveats**: divergences between course families (a2 vs b2), or instructor-specific patterns if you can detect them.

7. **Write `exam_analysis.md`** in Chinese (the course is taught in Chinese). Structure:

   ```markdown
   # <课程代码> 历年考试分析

   <生成日期>。基于 N 份试卷（列出年份+季节）。本分析仅作复习权重参考，每年试卷可能与历史有偏差。

   ## 试卷样本

   | 年份 | 季节 | 课程 | 题型分布 |
   | --- | --- | --- | --- |
   | ... | ... | ... | ... |

   ## 章节考点频次

   | 章 | 节/主题 | 出现年份 | 常见题型 | 估计权重 |
   | --- | --- | --- | --- | --- |
   | 10 | §5 幂级数 | 2021、2023、2024 | 求收敛半径、求和函数 | 中等 |
   | ... |

   ## 高权重重点（建议优先掌握）

   - <主题>：理由（出现 X/Y 年；常以 Z 形式考查）

   ## 易忽略但常考的细节

   - ...

   ## 边缘考点

   - ...

   ## 局限与注意

   - 样本中 b2 卷与 a2 卷的差异：<观察>
   - 近年趋势：<观察>
   - 未在样本中出现但教材重点：<提醒>
   ```

8. **PDF audit not applicable** — your output is markdown, not a PDF. But if you read any solution PDF, briefly note what you saw on at least one page to confirm the file was actually parsed (per the global `pdf.md` rule's "describe what you saw" principle).

9. **Tell the user**:
   - How many papers you read.
   - The top 3 high-leverage topics by frequency.
   - Any papers you could not parse (e.g. corrupted, missing).
   - That `course-split-section` and `course-init-prompt` will pick up `exam_analysis.md` automatically.

## Heuristics

- If the topic distribution shows a single year with wild divergence (e.g. one paper tests heavily on a chapter no others touch), flag it but do not weight it as a recurring topic.
- If solution PDFs reveal solution-method choices that differ from the textbook's standard approach, note this as "preferred solution style" — useful context for the init-prompt skill.
- Do not invent topics that you did not actually see on a paper. If the sample is too thin (≤2 papers), say so explicitly and recommend the user fetch more before relying on the analysis.

## Re-running

- Safe to re-run whenever new papers are added to `exam_papers/`. Overwrite the previous `exam_analysis.md` and tell the user what changed (which years are now included, whether the top-leverage list shifted).
