---
name: bib-verify
description: Bibliography authenticity check for academic papers — WebSearch every entry to confirm author, editor, translator, and year. Catches LLM fabrication. Fabrication hotspots are Chinese journal articles, edited volumes, secondary cites tacked onto primary sources, and translators of Chinese editions. Use after prose polish, before final delivery.
allowed-tools:
  - Read
  - Edit
  - WebSearch
  - WebFetch
  - Bash
metadata:
  trigger: Verifying bibliography entries on an academic manuscript before delivery
---

# bib-verify

Citations are the highest-risk surface for LLM-assisted writing. The prose-polish pipeline does **not** verify them — that's this skill's job. Run it separately, after polish, before final delivery.

This is the **hard gate**, not the only defense. `grounded-draft` works upstream to stop fabricated cites entering the draft at all (argument grown from a verified source ledger). When a draft came through `grounded-draft`, this skill's job narrows to clearing the `[retrieved · UNVERIFIED]` tier and confirming claim-fidelity against the ledger notes; when it didn't, run the full per-entry procedure below. Either way, do not skip this gate — grounding is soft-enforced and does not make verification optional.

## Why bibliography fabrication slips through

Polished prose looks authoritative. Plausible-sounding citations look authoritative. Both can be wrong in ways that pass a casual read but fail when someone tries to fetch the source. Common patterns:

- **Real author + invented title** — the author exists and writes on the topic, the cited title doesn't.
- **Real title + wrong year/edition** — usually off by 1–3 years; off by an edition is worse for page references.
- **Invented translators of Chinese editions** — the source is real, the translator is plausible-sounding but fictional.
- **Secondary cites laundered as primary** — "X (Year) argues..." where X discusses the idea but the line attributed to X is actually paraphrasing Y.
- **Edited volume confusion** — the editor's name swapped with the chapter author, or the volume title swapped with the chapter title.

## Verification procedure (per entry)

For each `\bibitem` / BibTeX entry / footnote:

1. **WebSearch the title verbatim** plus author surname. Quote the title to force exact-match.
2. If found, **WebFetch the canonical source page** (publisher, library catalog, JSTOR, CNKI, etc.) and verify:
   - Author name (and order, for multi-author)
   - Editor name (for edited volumes)
   - Translator name (for translated works)
   - Year of publication (and edition)
   - For chapters: chapter author + volume editor + volume title — all three
3. If not found in 2–3 queries, treat as **suspect**. Don't assume your search failed — assume the entry might be fabricated.
4. For suspect entries, do **one more targeted search** using different terms (author + topic, or partial title). If still nothing, flag for the author to confirm from their own notes.

Use `Read` on the source file (e.g. `main.tex` or `references.bib`) to walk through entries; use `Edit` to flag suspect ones in-place with a comment (`% UNVERIFIED — author to confirm`) rather than silently deleting.

## Fabrication hotspots (verify these first)

1. **Chinese journal articles** — especially those without a DOI. CNKI and Wanfang are the primary catalogs; cross-check both.
2. **Edited volumes** — the chapter-author-vs-volume-editor confusion is common. Confirm chapter author + volume editor + chapter title + volume title separately.
3. **Translators of Chinese editions** — the most fabrication-prone field. Verify against the publisher's page or a library catalog (e.g. NLC, Worldcat).
4. **Secondary cites attributed as primary** — if a citation supports a specific claim, fetch the actual source if accessible and confirm the claim is in it (not just the topic).
5. **Pre-print papers** — verify on arXiv / SSRN / bioRxiv directly. LLMs sometimes invent plausible-sounding pre-print IDs.
6. **Recent (last 12 months) citations** — LLM training-data cutoff issues mean recent works are more likely to be hallucinated or have wrong details.

## Output format

Produce a verification report:

```
✓ ref1   Smith, J. (2019). Title. Publisher.
✓ ref2   Wang, X. (2021). 标题. 期刊名, 卷(期), 起页–止页.
?  ref3   Li, M. (2018). Title. — TITLE NOT FOUND in 3 searches. Author to confirm.
✗ ref4   Chen, Y. (2020). Title. — Author exists, but title appears fabricated.
                                    Possibly meant: Chen, Y. (2019). Different Title.
```

- `✓` confirmed, all fields match
- `?` not found, requires author manual confirmation
- `✗` evidence of fabrication or significant error — propose the likely correct entry if one exists

## Edge cases

- **Print-only sources** (older books, theses not digitized): WebSearch may not find them. Library catalog (Worldcat, NLC, university library) is the fallback. If the author claims they have the physical copy, log it as confirmed-by-author and move on.
- **Self-citations**: the author owns the truth here. Still verify year and exact title against their other listed works for consistency.
- **Translations from non-English sources**: confirm both the original title and the translator. The translator field is the high-risk field.

## Limits — what this skill does *not* do

- It does not verify that the cited *content* supports the *argument* it's marshaled for. That's an editorial-substance check, separate from this authenticity check.
- It does not produce a correctly formatted bibliography. Use the journal's style guide for that; this skill only confirms the data is real.
- It does not silently fix entries. Suspect entries are flagged for the author to decide.

## When to invoke

- Any academic paper before final delivery.
- Whenever a citation list was assembled with LLM assistance (drafting, translation, or expansion).
- Whenever the author themselves suspects an entry might be wrong.
