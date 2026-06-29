# Teaching-style preferences (distilled from the user's refined packets)

These are the durable preferences the user has converged on across many `init_prompt.md` files, written and hand-edited over a full semester of two courses (高数 A2, 高代 A2). They are the *content* of the "Preferred teaching style" and "User's goal" sections — the part the user cares about most. `course-init-prompt`'s SKILL.md owns the structure; this file owns what goes inside it. Fold the relevant items into each draft, adapted to the section. Reuse the canonical Chinese phrasings at the bottom verbatim where they fit — the user has tuned that wording.

Many of these started as the user *correcting* a previous packet ("上个 packet 把题堆在最后，我不喜欢"). They are not stylistic garnish; each one is a lesson paid for. Do not drop them to save lines.

## The core stance

- **The user learns primarily from the assistant's explanation, not the textbook.** "我主要靠你的讲解学，课本只用来核对细节。" The theory must be explained until the user can follow it and derive the next step himself — not a dump of conclusions and unfamiliar symbols. The textbook is for cross-checking details (cite page numbers so he can).
- **"高效" (efficient) ≠ skipping explanation.** The 1-hour goal and "最简解法 / 不堆砌无关推广" must NOT be read as "skip the teaching." Motivation, definitions, and key derivations cannot be skipped — *that is the thing the user values most* and the line he has restated in packet after packet. What "efficient" actually prunes: off-syllabus generalizations and unrelated tangents, not the explanatory spine. Always pair the conciseness ask with this explicit caveat; "最简解法" alone has burned him.

## The standard teaching-style block (learn-a-new-section mode)

The recent learn-new-section packets carry a numbered list very close to the following. Treat it as the canonical menu; include the items that apply, in the user's own framing:

1. **动机先行 / 数学史式叙事 (motivation first).** Each concept: first say what problem it was born to solve, or which already-learned theory it naturally extends — *then* give the definition. Never open with the abstract definition / high-viewpoint conclusion / a new symbol cold (e.g. don't lead with "$V\otimes U$ 的定义和 $\otimes$ 符号").
2. **符号即拆、公式当场推 (unpack symbols, derive in place).** Every notation gets unpacked on first appearance; key results are derived on the spot, not handed over as a finished formula.
3. **自底向上 (bottom-up ladder).** Build from where the user can actually compute (a determinant, a concrete small Kronecker product, a specific bilinear form) up to the abstract statement. The explanation must be self-contained; cite textbook **page numbers** for every reference so he can verify.
4. **分段不塞满、循序渐进 (segment, don't cram).** Two phases. **Phase 1** — one or two paragraphs laying out the section's main thread (骨架), closing by hooking that skeleton onto a named concept already in the uploaded prior materials. **Phase 2** — multiple rounds, one or two examples each. **After each round, stop and ask whether the depth/pace is right before continuing. Do not cram a whole round into one message.**
5. **重要性看考试 + 知识体系，不看作业是否布置 (weight by exam + structure, not homework).** Decide teaching depth from exam relevance and the knowledge structure, never from whether a point appears in the homework. A section with **no assigned homework can still be a heavy exam topic** (e.g. 张量积: no homework, but newly in scope and already directly tested) — teach it to full exam depth regardless. Conversely, do not over-weight a point just because it happens to be assigned.
6. **真题随讲随挂、随讲随练，绝不堆到最后 (weave past papers into the matching round, drill as you go — never pile them at the end).** This is an explicit, emphatic correction the user made. Each related past-paper problem goes **into the round whose topic it matches**: quote its statement inline, drill it on the spot, tag its paper family — do not save all the problems for a single block after the theory. When there is no homework, the past-paper problems *are* the practice, so this matters even more. (Carry through every guardrail from the SKILL's "Past-paper references": family tag on every citation, never pass a 同源/B-卷 problem off as direct A-卷 evidence, reference only problems actually attached, never invent a 题号.)
7. **每轮开讲前先想怎么讲得更好 (reflect before each round).** Before each round the assistant should think about how to teach it best — give the skeleton first when the content is heavy, pay off any foreshadowing it planted, anchor the abstract content into one sentence, and end by naming the design intent of the problem. It may proactively improve its own approach.
8. **公式渲染 (LaTeX that renders on claude.ai).** See "Rendering rules" below — these are concrete and reusable across every section.

Phases 1/2 and the "stop and check pace" rhythm are already in the SKILL's required structure; this list adds the *texture* the user wants inside them. Don't bolt these on as separate ceremony — weave them into the two-phase teaching-style section.

## Connecting to prior learning — now via claude.ai **project files**, including past cheatsheets

The user's workflow has moved to claude.ai **Projects**: prior chapters' handwritten notes *and the cheatsheets/learning outputs from earlier packets* (e.g. `高代A2_Packet2_考前速查.pdf`) live in the conversation's **项目文件 (project files)**, not only as per-message attachments. So:

- Phrase the connect-to-prior instruction around "我**已上传文件**里确实有的概念 / 项目文件里的笔记和前几个 packet 的 cheatsheet", and tell the session to **open those files** during teaching.
- Keep the grounding guardrail exactly as the SKILL states it: connect only to concepts actually in the uploaded files; if a needed concept isn't there, say so plainly rather than assuming the user's background. The session is still stateless — it knows only what is uploaded.
- When the section has obvious bridges to specific prior concepts, list them explicitly as named hooks (the recent packets spell out 3–4 bridges, e.g. "多重线性映射在 $r=2$ 时就是 §10.1 的双线性函数", "$AX-XB^{\mathrm T}$ 的特征值接 §10.5 谱理论"). Concrete named bridges beat a generic "回顾旧知识".

## Closing: tie the thread into one sentence

End the prompt by asking the assistant, after all rounds, to compress the section's through-line into **one sentence** for pre-exam quick recall (e.g. "把『多重线性映射 → 张量积 → Kronecker 积 → 与谱理论的联系』串成一句话"). The user uses these as 考前速记.

## Review / drill mode (finalterm, ztf-notes-style packets)

Some packets are not "learn a new section" but **exam review / problem-set drill** over already-studied chapters. They share the core stance above but swap the rhythm:

- **Answer-check first, every round.** "每轮开头先报上一题（或上一份卷）的答案让我核对，再往下讲。" The user does the problem himself first; the assistant opens each round by giving the previous problem's answer for him to check, then continues; he asks if he has questions. "这条一直守住。"
- **只讲答案不讲方法等于白做.** For every drilled problem, surface the method / the annotation behind the solution, not just the final answer.
- **Mention the small but reusable textbook conclusions too**, not only the headline example problems — e.g. $\int_0^{+\infty}\frac{\sin x}{x}\,dx=\frac\pi2$, $\int_0^{+\infty}e^{-x^2}\,dx=\frac{\sqrt\pi}2$. Past papers reuse these constantly.
- **Treat the abstract language as real content**, not a computational shortcut to wave past. Inner product $(f,g)$, norm, 函数空间, 单位正交系, "傅里叶系数 = 正交基上的坐标" — the user explicitly got burned ("吃过亏") when an earlier class treated this as a mere tool, and struggled on the theory problems. Teach it from scratch.
- A review packet is usually driven by a **master outline file** (`review_outline.md` / `gap_analysis.md` + `problem_map.md`) that is the spine of the session; point the prompt at it and follow its staging.

## Rendering rules (claude.ai LaTeX/markdown — concrete, reuse verbatim)

These are hard-won and apply to essentially every prompt; include them:

- Use LaTeX that **claude.ai renders correctly**.
- **Do not put `·` or `\cdot` inside `\text{...}`.** For a centered dot between Chinese words write `\text{甲}\cdot\text{乙}`, or just use `、`. Never use `•`.
- **In markdown tables, write absolute value / norm as `\lvert \rvert` and `\lVert \rVert`, never the bare pipe `|`** — a literal `|` collides with the markdown table column separator and shreds the table. (This bites in any table with absolute values; it is the single most repeated rendering fix.)

## Practical constraint worth stating when relevant

claude.ai caps a single conversation at **~100 pages including PDF pages**. When a packet's attachments would blow past that, the user splits the work across sessions (the ztf review set became three sessions). If you are assembling a packet whose attachments are large, flag the limit rather than producing an un-uploadable bundle.

## Canonical Chinese phrasings to reuse

Drop these in where they fit; the user has already tuned them.

```text
Opener:            下面是背景和我对你的期望。
Stance:            我主要靠你的讲解学，课本只用来核对细节。所以理论必须讲到我能听懂、能自己往下推，而不是抛结论和我没见过的符号。
Efficiency caveat: 要高效——别堆砌与考点无关的推广和旁支；但「高效」不等于省略讲解：动机、定义、关键推导一个都不能跳。
Phase-2 pacing:    分多轮深入，每轮一两个例子，讲完一轮就停下来问我深度/节奏对不对，再继续。不要一条消息塞满一整轮。
Past-paper weave:  把每道真题放进它最相关的那一轮里，当场引出题面、当场推演，而不是讲完全部理论再统一刷题。
Family-tag guard:  引用时务必带卷别标签，直接 A 卷样本很薄，绝不能把 B 卷旁证当成 A 卷的直接命题依据；只引用附件里真有的题，没有就直说，不要编题号。
Rendering:         用 claude.ai 能正确渲染的 LaTeX——不要把 · 或 \cdot 放进 \text{...} 里；别用 •；markdown 表格里的绝对值/范数一律用 \lvert \rvert、\lVert \rVert，不要用竖线 | 。
Drill answer-check: 每轮开头先报上一题的答案让我核对，再往下讲——我会自己先做，你报完答案直接接着讲，我有问题再问。这条一直守住。
Closer:            请从第 1 步开始。
```
