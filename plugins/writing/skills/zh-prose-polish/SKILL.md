---
name: zh-prose-polish
description: The locked polish pipeline for Chinese prose — sequence drafts through consult-zh, humanizer-zh, and de-AI-writing in a fixed order, with measurable AI-trace targets (破折号 ≤ 6 per 5000 字, zero 否定式排比, zero 三段式 forced groupings). Use after producing a Chinese draft (essay, paper, translation) and before final delivery.
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
metadata:
  trigger: Polishing a Chinese-language draft before delivery; coordinating consult-zh + humanizer-zh + de-AI-writing
---

# zh-prose-polish

Coordinator skill for finishing Chinese prose. The individual skills (`consult-zh`, `humanizer-zh`, `de-AI-writing`) each do one thing well; this skill enforces the **order** they're applied in and the **metrics** the final draft has to hit.

The order is not negotiable. Running them out of order produces worse output for documented reasons (see §3).

## 1. The locked pipeline

```
draft  →  consult-zh  →  humanizer-zh  →  de-AI-writing  →  final
         (argument)     (surface AI tells)   (二元对照, 路标词, 模板)
```

1. **draft** — section-by-section, in the author's own voice. Don't pre-polish; let the next steps do their jobs.
2. **`consult-zh`** — argument critique + phrasing via DeepSeek in a draft+critique round. Fix structural and idiomatic issues *first*, before any cosmetic scrub. Multi-round via `--session` if needed.
3. **`humanizer-zh`** — surface-level AI-trace scrub against the Wikipedia "Signs of AI writing" pattern set: 破折号 overuse, 三段式法则, 否定式排比, AI 高频词, 模糊归因, etc.
4. **`de-AI-writing`** — author-voice retention pass against the good-writing 文风底座. Strips the 二元对照 template (`不是…而是`), 路标词 budget enforcement, 分析师讲解 cadence removal.
5. **final user read** — the author has the last say. Never skip this step.

## 2. Measurable targets (1500–5000 字 final text)

These are the hard checks before declaring polish done. Run `grep`/`wc` to verify, don't eyeball.

| Metric | Target | How to check |
|---|---|---|
| `而是` (二元对照 hinge) | **0** (豁免 ≤ 1 if原文自然) | `grep -o 而是 file.md \| wc -l` |
| 否定式排比 (`不仅…而且`, `不只是…更是`) | **0** | scan for the two-clause negation patterns |
| 三段式法则 (forced 3-item groupings) | **0** forced | check enumerations: was it really 3, or padded to 3? |
| 破折号 `—` | ≤ 6 per 5000 字 | `grep -o "—" file.md \| wc -l` |
| 全角冒号 `：` in body | 0–2, only as 直接引语 prompt | `grep -nE "：" file.md` |
| 二人称 `你/你会` | ≤ 1, only as 归谬推演 | `grep -nE "你会?" file.md` |
| 路标词 (`换句话说/事实上/值得注意/总之/与此同时/更关键/更要命`) | ≤ 2 合计 | grep alternation |
| AI 高频隐喻 (噪音/信号/底色/光谱/滤镜/解药/土壤/基因/拼图/镜像/路径/标尺/切面/透镜/窗口/缩影) — 比喻义 | **0** | grep + manual disambiguation (字面义 exempt) |
| 降维引导语 (`说白了/本质上/归根结底/简单来说/换个角度看`) | ≤ 1 合计 | grep alternation |
| 极值判断 (`最…的地方在于/真正…的是/…之处在于`) | ≤ 1 合计, must follow with verifiable fact | grep alternation |

Numbers scale roughly with length — for 1500 字, halve the budgets; for 5000 字, use as-is.

## 3. Why the order matters

- **`consult-zh` first**: argument-level issues distort word choice downstream. Polishing surface tells before fixing a wobbly thesis means you'll polish text you'll throw away.
- **`humanizer-zh` before `de-AI-writing`**: humanizer-zh handles the *generic* AI tells (破折号, 三段式, 否定式排比) that apply to any LLM output. de-AI-writing then enforces the *author-specific* style (作者文风底座). Running de-AI-writing first means you fight against generic tells the humanizer would have caught with less collateral damage to voice.
- **Author final read last**: every previous step is an automated or model-assisted pass. The author's judgment outranks all of them — never skip.

## 4. Common deviations and what they cost

- **Skip `consult-zh` entirely** (drafted by you, no DeepSeek critique): acceptable for short pieces or when DeepSeek is unavailable. Note in delivery that it was skipped.
- **Skip `humanizer-zh`** (jump straight to de-AI-writing): leaves the generic AI tells in. de-AI-writing's checklist overlaps but is tuned for voice, not for the Wikipedia pattern set. Don't skip unless the draft was hand-written from the start.
- **Skip `de-AI-writing`**: leaves the prose voice-flat. Acceptable for translation tasks where structural保真 matters more than voice (use `de-AI-writing`'s translation mode instead — see its §5).
- **Reverse the order**: don't. The output gets visibly worse — author voice fights cosmetic cleanup, structural issues get rewritten by the AI-trace scrubber, etc.

## 5. Bibliography is separate

The polish pipeline **does not verify citations**. Bibliography authenticity is its own concern with different failure modes (fabricated authors, wrong years, invented translators). After polish, invoke `bib-verify` separately before final delivery.

## 6. Reporting back

When delivery time comes, briefly note in 1–3 lines:

- Which pipeline steps ran (full sequence, or what was skipped and why)
- How many critique rounds in `consult-zh`
- Final metric scores (the table in §2)
- Anything the author should review manually that the pipeline couldn't decide for them

Don't paste the intermediate drafts — the user wants the final, not the process.

## When to invoke

- Polishing a Chinese-language draft of any length before delivery (essays, papers, translations).
- Coordinating multiple writing skills in sequence (so the order is enforced).
- Before submitting any Chinese academic text where AI-trace detection is a risk.
