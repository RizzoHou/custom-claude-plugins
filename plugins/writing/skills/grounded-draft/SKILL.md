---
name: grounded-draft
description: Source-grounded drafting for essays and papers — stop Claude inventing citations by growing the argument out of a verified source ledger instead of back-filling cites under pre-written claims. The argument and the evidence co-evolve in a loop; drafting may cite only ledger IDs, and any unsupported claim is revised, downgraded, or flagged — never fabricated. Use before drafting any academic text that makes sourced claims; hand off to zh-prose-polish then bib-verify.
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - Bash
metadata:
  trigger: Drafting an essay or paper that makes externally-sourced claims, especially when fabricated references have been a problem
---

# grounded-draft

The skill that runs *before* a draft exists. It exists to kill one failure: **references invented out of nowhere**, which collapse the essay's evidence chain.

## 1. Why fabrication happens (and why a stricter checker won't fix it)

Claude fabricates citations because of the *order* it works in: it forms the argument first — from its "what a strong essay sounds like" prior — then **back-fills** citations to prop up claims already written. When no real source says the thing, the pressure to support the pre-written claim produces an invented one.

A post-hoc checker like `bib-verify` catches the fake *after* the argument already leans on it — by then fixing it means rewriting the argument. The fix has to move upstream: the argument must be **answerable to the sources**, grown out of what real sources actually support, not imposed on them and back-filled.

So this is not a pipeline (form argument → go find support). A fixed argument sent out to "find support" is motivated retrieval — it cherry-picks, and fabricates when nothing fits. It is a **loop** in which the argument and the evidence co-evolve.

## 2. The two artifacts

Keep both as files in the working directory; they co-evolve every pass.

### `ledger.md` — the closed set of real sources

Nothing may be cited that is not here. Two tiers:

```
### S1 [user]
Smith, J. (2019). *Title*. Publisher.
- note: "<exact quote>" (p.42) → supports: X causes Y under condition C.
- note: "<exact quote>" (p.88) → supports: the effect reverses above threshold T.

### S2 [retrieved · UNVERIFIED]
王芳 (2021). 标题. 期刊名, 卷(期), 起–止页.  url: https://…
- note: "<quote / close paraphrase with locator>" → supports: Z holds in the Chinese case.
```

- `[user]` — tier-1, from the author's own corpus (BibTeX / Zotero export / folder of PDFs / reading list). The author owns their truth.
- `[retrieved · UNVERIFIED]` — tier-2, found by web retrieval this session. **Provisional until `bib-verify` clears it.**
- A source with **no note is not in the ledger.** A note is the specific claim the source supports plus a quote/locator — that is what makes the eventual citation faithful, not just authentic.

### `argument.md` — the claim map

```
THESIS (provisional): <one sentence>

1. <sub-claim>                         [sourced → S1, S3]
   1a. <sub-sub-claim>                 [reasoned]
2. <sub-claim>                         [open → retrieve or weaken]
3. <sub-claim>                         [sourced → S2 (UNVERIFIED)]
```

Every node is tagged with one of three kinds — because **not every claim should be cited**:

- **sourced** — asserts an external fact; needs a ledger note. *This is the only fabrication-risk surface.*
- **reasoned** — your own inference, synthesis, or interpretation; uncited *by design*. Legitimately carries no citation.
- **open** — currently unsupported. Must be resolved before drafting (see §4). Never left to be fabricated.

## 3. The loop

1. **Seed.** Write a *provisional* thesis (a hypothesis, marked revisable). Ingest the author's corpus into `ledger.md`. Sketch `argument.md` both top-down (what the thesis needs) **and** bottom-up (what the corpus already says — let real notes suggest claims you didn't plan).
2. **Pick the weakest node** — the most `open`, or the `sourced` node with the thinnest note.
3. **Read one source** for it (corpus first, else retrieve — §5). Extract notes into the ledger with quote + locator.
4. **Reconcile the map to the notes.** This is the step that closes the coupling: confirm the claim, **weaken it to what the note actually supports**, split it, drop it, or **add a claim the note revealed**. Re-tag nodes. The argument bends to the evidence here — not the reverse.
5. **Loop** until convergence (§6), then draft (§7).

## 4. The four moves for an `open` claim

When a claim wants support it doesn't have, exactly four moves are allowed — fabrication is not one:

1. **Retrieve** — go find a real source, read it, add a note. (Back to loop step 3.)
2. **Revise to fit** — rewrite the claim down to what the available evidence *does* support. The deepest move, and the one that distinguishes this from a pipeline: the claim changes, not just its citation.
3. **Downgrade to reasoned** — if it is genuinely your own inference, re-tag it `reasoned` and stop pretending it is externally sourced. (Don't abuse this to launder unsupported assertions — only for real synthesis.)
4. **Weaken or drop** — hedge it to a defensible scope, or cut it.

If none can be done now and the author will supply the source offline, leave a **visible** `[CITATION NEEDED: <claim>]` placeholder in the draft. A placeholder is an honest hole; an invented cite is a lie. Never trade the first for the second.

## 5. Retrieval (tier-2)

Only when the corpus has no note for a node:

- WebSearch for a real source; **WebFetch and actually read enough to extract a note** — a source you can't read the relevant passage of does not get a note, and without a note it is not in the ledger.
- Chinese sources: CNKI / Wanfang are the catalogs; the fabrication hotspots `bib-verify` knows (Chinese journals without DOI, edited volumes, translators of Chinese editions, secondary-cites-as-primary) apply here too.
- Paywalls cap grounding: if you can confirm a source *exists* but cannot read the passage, do **not** write a note asserting what it says. Either find an accessible source or mark the claim `open`.
- Every retrieved entry is `[retrieved · UNVERIFIED]` until `bib-verify` clears it.

## 6. Convergence — ready to draft when

- Every `sourced` node has at least one real ledger note under it.
- Every `open` node has been resolved by one of the four moves in §4.
- The thesis is consistent with the notes (it may have moved from its provisional seed — that is success, not drift).

## 7. Drafting — the hard contract

Now write. Because the argument was grown from the ledger, "cite only from the ledger" is natural rather than restrictive.

1. **Every in-text citation references a ledger ID.** No source is born during drafting — if you reach for one not in the ledger, stop and run a loop pass (§3), don't invent.
2. **A claim may not assert more than its note supports** — claim-fidelity, the gap `bib-verify` explicitly cannot check. Track the note; do not round it up.
3. **Unsupported-but-wanted source → `[CITATION NEEDED]`, never invented.**
4. **`[retrieved · UNVERIFIED]` cites stay flagged** through to the `bib-verify` gate.

## 8. The thesis-pinning knob

- **Provisional (default):** evidence may rewrite the top-level thesis. Most faithful, most fabrication-proof.
- **Pinned:** the author is arguing an assigned or pre-decided position. The headline is fixed, **but the sub-claims are still answerable to the evidence** — unsupported ones become visible holes or get weakened, never fabricated. The evidence always gets to push back on the sub-claims; only whether it can move the headline is the author's switch.

Ask which mode if it isn't obvious from the request.

## 9. Handoff

This skill ends at a grounded draft, not a finished one. Hand off:

1. **`zh-prose-polish`** — the locked prose pipeline (consult-zh → humanizer-zh → de-AI-writing).
2. **`bib-verify`** — the **hard gate** before delivery, focused on every `[retrieved · UNVERIFIED]` entry. The ledger notes also make `bib-verify`'s claim-fidelity check tractable: each cite already carries the passage it was supposed to support.

The contract: this skill shifts the load left so most fabrication never enters the draft; `bib-verify` stays as the gate because the ledger constraint is soft-enforced (a discipline, not a sandbox) and models drift under generation pressure. Grounding does **not** make verification optional.

## 10. Limits — what this skill does not do

- It does not *guarantee* zero fabrication — it makes it costly and visible. `bib-verify` is still required.
- It does not format the bibliography to a journal's style guide.
- It does not invent access: behind a hard paywall with no accessible source, the honest output is an `open` claim or a placeholder, not a confident cite.

## When to invoke

- Before drafting any essay or paper that makes externally-sourced claims.
- Whenever fabricated or mismatched references have been a problem before.
- When the author has a corpus (BibTeX / Zotero / PDFs / reading list) to ground in, or a topic that needs sources retrieved — or both.
