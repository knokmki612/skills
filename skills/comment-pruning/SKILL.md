---
name: comment-pruning
description: Prune comments, developer docs, and the documents a task produces (reports, specs, plans) of text that no longer earns its reading cost — history narration git already records, working notes and superseded drafts left in a deliverable, drifted references, restatements of facts tests or schemas enforce, and blocks that overload the reader. Classify every passage (keep / rewrite / relocate / delete), reshape overloaded units to per-unit load budgets, guarantee the information survives in at least one durable place, and verify substance is unchanged with an annotation-only diff and the project's checks. Use when comments have piled up over iterations of agent or human work, when a file reads like a changelog, when a deliverable still carries the trail of how it was written, when docs have grown too long to read, after a multi-round change settles and before finalizing a PR or handing a document over, or when asked to clean up, tidy, prune, or organize comments or documentation.
license: CC-BY-4.0
---

# Comment Pruning

## Why

Iterative work — especially agent-assisted — deposits a sediment in whatever
it writes: code comments, docs, and the deliverable documents themselves. Each
decision, review response, migration, and abandoned draft felt worth recording
at the moment it was made. But git log, blame, PRs, and the conversation
already record history. An annotation's job is to help the reader of the
**current** text; judge it by "reader's effort saved at that spot" vs "noise +
drift cost". A passage that retells a story the history already tells fails
that test on both sides. So does one that points at code that has since moved,
and — less obviously — a block whose every fact is individually justified but
whose aggregate exceeds what a reader can hold: volume is a defect in itself,
so budgets apply per reading unit, not only per fact.

Deleting text is cheap; destroying information is not. The invariant this skill
protects is that **every piece of information survives in at least one durable
place** — the code itself, the current docs, or git history (including the
pruning commit's own message). Pruning is a *move*, never a *loss*.

## Scope — what counts as an annotation

An annotation is any passage this skill judges: a comment, a doc paragraph,
or a sentence in a deliverable that carries process instead of state. Either
of two tests puts text in scope:

- **Addressed to developers.** Code comments in any syntax (`//`, `#`,
  `/* */`, `{# #}`, `<!-- -->`, docstrings, template comments) and developer
  docs (README, `docs/`, contributing/setup notes, architecture notes).
- **Produced or revised in this task.** Any document the task delivers — a
  report, a spec, a plan, an analysis, an article — whoever its reader is.
  The failure modes are the same: a superseded draft left beside its
  replacement, a working note, the trail of how the text was written.

Out of scope — never prune:

- **Pre-existing end-user content the task did not revise.** In a
  content-driven repo (static site, CMS-backed), `.md` files may be published
  prose; "Compared to conventional products…" in a product article the task
  did not touch is content, not annotation.
- **Machine-read directives**: shebangs, `eslint-disable`, `@ts-expect-error`,
  `noqa`, pragmas, front matter the build consumes, editor/tooling markers.
- **License and copyright headers.**

When one file mixes classes (machine-read front matter, a published body the
task did not revise, developer comments inside it), only the in-scope parts
are candidates.

Boundary with `docs-restructuring`: this skill fixes what fits inside the
existing structure. When the structure itself has failed — headings that no
longer predict content, sections answering several reader questions at once —
run the `docs-restructuring` skill first, then prune.

## Execution model

Two phases, run in order. Never merge them: Phase 1 writes the
classification table down before Phase 2 touches anything, and Phase 2
executes that table and nothing else. The table is reproduced in the report.

```
Phase 1  ANALYZE (read-only)  →  classification table
Phase 2  EXECUTE the table  →  VERIFY annotation-only diff + checks  →  report
```

## Phase 1 — Inventory and classify

Fix the scope first: the current branch's touched files
(`git diff --name-only <base>..HEAD`), a directory, the whole repo, or the
documents produced in this task. When the request leaves it open, take the
narrowest reading that covers the request and name the chosen scope in the
report.

Give every annotation in scope exactly one verdict:

| Verdict | Criterion | Action |
| --- | --- | --- |
| **KEEP** | A guard: a point that would puzzle a reader of the implementation, a DO-NOT / pitfall, a rationale that cannot be read off types, signatures, or nearby code. A convention stated once at its enforcement point. A heading line over a non-trivial block. In a deliverable: a conclusion, a fact, a recommendation, and a comparison of alternatives where the document exists to record the decision. | None. |
| **REWRITE** | A living fact wrapped in historical narration ("since #123 we now use X"); a conclusion stated as a trail ("at first A, then changed to B"); a reference pointing at code that moved or was renamed; a block or list packing more facts than a reader can hold. | State the fact in present tense; fix the pointer onto a stable anchor; split, regroup, or pointer-ize down to one concern per unit. |
| **RELOCATE** | Rationale that still matters to design or usage but belongs in docs (README, architecture notes) and is not yet there. A rejected alternative or working note that still matters but not to the document's reader. | Move it to the right doc — or, for a deliverable, to the commit message, PR description, or hand-over report — then delete it here. |
| **DELETE** | History or provenance git already records; a paraphrase of the adjacent code; a leaf-level echo of a convention stated elsewhere; commented-out code. In a deliverable: working notes ("revisit later"), superseded drafts kept beside their replacement, "update:" or addendum notes whose content the text above already carries. | Delete. |

**Load budgets.** One annotation = one concern. Comment blocks stay within
~4 lines unless a guard genuinely needs more; doc lists stay within ~5 sibling
bullets, regrouped by reader intent (understand / not break / do / look up)
when they grow past that; facts already enforced by tests, schemas, or lint
rules become one-line pointers to the enforcement point, never restatements.

**The provenance test.** Before marking a history passage DELETE, confirm the
story is actually recoverable — `git log --follow -p -- <file>`, `git blame`,
a referenced PR. If it is not (the decision lived only in a chat and the
passage is its sole record) **and it still matters**, the verdict is RELOCATE —
or record it in the pruning commit's message body, which turns the deletion
itself into the durable record. For a deliverable, the hand-over report plays
the same role: it names what was dropped and where it now lives.

Detailed signals (narration, drift, working notes, volatile anchors), load
budgets, worked examples, and edge cases (TODOs, commented-out code, doc
comments consumed by tooling): see
[references/classification.md](references/classification.md).

Write the plan down as a table — `file:line`, the annotation (truncated),
verdict, and *where the information lives afterwards*. Phase 2 executes this
table and the report reproduces it. Do not edit anything in Phase 1.

## Phase 2 — Execute and verify

1. **Apply the Phase 1 verdicts.** Annotations only — never change code, and
   never change a deliverable's conclusions, figures, or recommendations, in
   the same commit. If pruning exposes a code smell or a contradiction between
   conclusions, report it; do not fix it here.
2. **Annotation-only diff check.** Review `git diff` and confirm every hunk
   touches only comments and docs; in a deliverable, that every hunk removes
   or reshapes process text and leaves each conclusion, figure, and
   recommendation in place. Where comment syntax can reach build output (HTML
   `<!-- -->` in templates), prove the output is unchanged: build before and
   after into separate directories and diff them.
3. **Run the project's checks.** Consult the task manifest (`package.json`
   scripts, `Makefile`, etc.) and run format/lint/test. Do not proceed with
   failures.
4. **Commit.** The message summarizes what was pruned and names where each
   relocated fact now lives. Rationale that was deleted and is recorded nowhere
   else goes in the message body — the commit message is the relocation target
   of last resort. An unversioned deliverable has no commit; the hand-over
   report carries the same information.

## Report

- Counts per verdict and files touched.
- Where each RELOCATE landed; for a deliverable, what was dropped from it and
  where that now lives, so the reader can ask for it.
- Confirmation that the diff was annotation-only, checks passed, and (where
  applicable) build output was byte-identical.
- **Out-of-scope observations** — candidate file deletions, suspected bugs,
  code smells, structural failures for `docs-restructuring`. Reported, never
  acted on here.

## Timing — when to run

- After a multi-iteration change settles — before opening or finalizing a PR —
  so review reads the code, not the story of writing it.
- Before handing over a document the task produced, once its content has
  converged — so the reader gets the conclusion, not the path to it.
- When a touched file reads like a changelog.
- On explicit request.

Not after every commit, and not mid-implementation or mid-draft: pruning
while the text is still moving churns the very notes that are guiding the
work.

## Never

- Delete information that would then survive nowhere — not in code, docs, or
  git history including the pruning commit's message.
- Mix behavior changes, or changes to a deliverable's conclusions, into a
  pruning commit.
- Prune pre-existing end-user content the task did not revise, license
  headers, or machine-read directives.
- Execute a verdict the Phase 1 table does not carry, or edit during Phase 1.
- Report success while the diff touches non-annotation lines or checks fail.
