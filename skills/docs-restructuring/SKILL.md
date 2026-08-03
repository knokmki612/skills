---
name: docs-restructuring
description: Restructure developer documentation whose structure has failed — the heading tree no longer predicts content, one list mixes concepts with invariants and procedures, half a document sits under an unrelated parent, or the same fact is maintained in several places. Rebuild the document around the four reader questions (understand / not break / do / look up), give every fact exactly one canonical home with pointers elsewhere, and verify with a move map that no passage is lost and no inbound link breaks. Structure only — per-unit tightening of comments and sentences is the comment-pruning skill's job; run it after this one. Use when a README or docs set has grown past navigability, when readers can't find things they know are written down, or when asked to reorganize, restructure, or split documentation.
license: CC-BY-4.0
---

# Docs Restructuring

## Why

Volume hurts only as much as structure fails. A reader who can jump straight
to the answer never meets the other 250 lines; a reader whose heading tree
lies to them must read linearly, and then every line costs. Structural failure
has recognizable shapes: a large subtree filed under an unrelated parent, one
bullet list interleaving concepts, invariants, procedures, and vendor
constraints, the same fact maintained in three places drifting apart.

The fix is placement, not prose. Developer documentation answers four
questions — *understand* (concepts, why it is designed this way), *not break*
(invariants, contracts), *do* (procedures), *look up* (reference data) — and a
reader arrives with exactly one of them. Structure serves the reader when each
section answers one question and the heading names it. See
[references/intent-model.md](references/intent-model.md) for the model, the
defect catalog, and target patterns.

Two invariants protect the operation: **every passage of the old document is
accounted for** (moved, pointer-ized, or explicitly retired — never silently
dropped), and **content moves verbatim** — reshaping sentences is the
`comment-pruning` skill's job, kept out of this diff so review can verify
moves instead of re-reading everything.

## Scope — and the boundary with comment-pruning

In scope: developer docs — README, `docs/`, contributing/architecture/setup
notes — their heading trees, section placement, splits and merges, and where
each fact canonically lives.

Out of scope:

- **Per-unit tightening** — shortening blocks, fixing drifted references,
  deleting history narration: that is `comment-pruning`. The boundary rule:
  *if it can be fixed inside the existing structure, prune; if the structure
  itself must change, restructure.* For a full overhaul run this skill first,
  then prune.
- **Code comments** (pruning's territory), **end-user content**, and
  generated documentation.

## Execution model

Two phases separated by an approval gate. Never merge them.

```
Phase 1  ANALYZE (read-only)  →  target tree + move map  →  ⛔ USER APPROVAL
Phase 2  EXECUTE (move verbatim)  →  VERIFY move map + links  →  report
```

## Phase 1 — Inventory and design

1. **Map the current structure.** Heading tree with per-section line counts.
   Oversized subtrees, mis-leveled parents, and monoliths show up here.
2. **Classify every passage** by the question it answers (understand / not
   break / do / look up). Passages answering several at once are split points.
3. **Decide each fact's canonical home.** Precedence: the code itself → an
   enforced check (test, schema, lint rule) → exactly one doc section.
   Everything else becomes a one-line pointer to the home. A fact a test
   already enforces is *stated* only at the test; the doc says what is
   forbidden and which check enforces it.
4. **Design the target tree** against the patterns and budgets in
   [references/intent-model.md](references/intent-model.md) (intent-pure
   sections, heading depth ≤ 3, sibling budgets, entry-point README).

Present the plan in this fixed format, then **stop for approval**:

```
CURRENT TREE (with line counts)      TARGET TREE

MOVE MAP
<old section / passage>  →  <destination | pointer to <home> | RETIRE (reason, where the information survives)>
...

Handed to comment-pruning afterwards: <sections needing per-unit tightening>
```

## Phase 2 — Execute and verify

1. **Move verbatim.** Cut and paste passages; write new glue only for heading
   lines and one-sentence pointers. Resist every temptation to "improve"
   sentences in passing — flag them for the pruning pass instead.
2. **Verify the move map** ⛔ completion gate: every heading and passage of
   the old document has a destination that matches the approved map. Nothing
   is dropped that the map did not explicitly retire.
3. **Verify inbound links.** Search the whole repo — docs, code comments,
   templates, config — for the old paths and anchors and update them.
4. **Run the project's checks** (formatter, markdown lint, link checkers if
   present) per its task manifest.
5. **Commit**, with the move map in the message body — it is the durable
   record of where everything went.

## Report

- Before/after heading trees and per-section line counts.
- The executed move map, including retirements and their justification.
- Inbound links updated (count and locations).
- What was handed to `comment-pruning` for per-unit tightening.
- Out-of-scope observations — content errors, suspected bugs, candidates for
  deletion. Reported, never acted on here.

## Timing — when to run

Rarely — this is structural surgery, not routine grooming:

- A reader (or agent) failed to find something that was written down.
- A document's heading tree stopped predicting its content, or one section
  answers several reader questions at once.
- A doc has grown monolithic and needs splitting, or duplicated facts have
  started to drift.

Routine tidying after a settled change is `comment-pruning`, not this.

## Never

- Drop a passage the approved move map did not explicitly retire.
- Rewrite prose while moving it — moves stay verbatim; tightening belongs to
  `comment-pruning`.
- Mix code changes, or content rewrites, into a restructuring commit.
- Leave an inbound link pointing at a heading or file that no longer exists.
- Restructure end-user content or generated docs.
- Skip the approval gate between Phase 1 and Phase 2.
