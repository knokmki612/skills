---
name: comment-pruning
description: Prune comments and developer docs that narrate development history — decision provenance, migration notes, "previously X, now Y" — whose information git history already records, keeping only annotations that help a reader understand the current code (guards, pitfalls, conventions, usage). Classify every annotation (keep / rewrite / relocate / delete), guarantee the information survives in at least one durable place, and verify behavior is unchanged with an annotation-only diff plus the project's format/lint/test checks. Use when comments have piled up over iterations of agent or human work, when a file reads like a changelog, after a multi-round change settles and before finalizing a PR, or when asked to clean up, tidy, prune, or organize comments or documentation.
license: CC-BY-4.0
---

# Comment Pruning

## Why

Iterative development — especially agent-assisted — deposits a sediment of
comments: each decision, review response, and migration felt worth recording at
the moment it was made. But git log, blame, and PRs already record history. A
comment's job is to help the reader of the **current** code; judge it by
"reader's effort saved at that spot" vs "noise + drift cost". A comment that
retells a story git already tells fails that test on both sides.

Deleting text is cheap; destroying information is not. The invariant this skill
protects is that **every piece of information survives in at least one durable
place** — the code itself, the current docs, or git history (including the
pruning commit's own message). Pruning is a *move*, never a *loss*.

## Scope — what counts as an annotation

In scope — text addressed to **developers**:

- Code comments in any syntax: `//`, `#`, `/* */`, `{# #}`, `<!-- -->`,
  docstrings, template comments.
- Developer docs: README, `docs/`, contributing/setup notes, architecture notes.

Out of scope — never prune:

- **End-user content.** In a content-driven repo (static site, CMS-backed),
  `.md` files may be published prose. "Compared to conventional products…" in a
  product article is content, not annotation. The test is *who consumes it*.
- **Machine-read directives**: shebangs, `eslint-disable`, `@ts-expect-error`,
  `noqa`, pragmas, front matter the build consumes, editor/tooling markers.
- **License and copyright headers.**

When one file mixes both (front matter + published body), only the
developer-addressed parts are candidates.

## Execution model

Two phases separated by an approval gate. Never merge them.

```
Phase 1  ANALYZE (read-only)  →  classification table  →  ⛔ USER APPROVAL
Phase 2  EXECUTE  →  VERIFY annotation-only diff + checks  →  report
```

## Phase 1 — Inventory and classify

Fix the scope first, confirming with the user if ambiguous: the current
branch's touched files (`git diff --name-only <base>..HEAD`), a directory, or
the whole repo.

Give every annotation in scope exactly one verdict:

| Verdict | Criterion | Action |
| --- | --- | --- |
| **KEEP** | A guard: a point that would puzzle a reader of the implementation, a DO-NOT / pitfall, a rationale that cannot be read off types, signatures, or nearby code. A convention stated once at its enforcement point. A heading line over a non-trivial block. | None. |
| **REWRITE** | A living fact wrapped in historical narration — "since #618 we now use X" carries a real rule inside a story. | State the fact in present tense; drop the story. |
| **RELOCATE** | Rationale that still matters to design or usage but belongs in docs (README, architecture notes) and is not yet there. | Move it to the right doc, then delete it here. |
| **DELETE** | History or provenance git already records; a paraphrase of the adjacent code; a leaf-level echo of a convention stated elsewhere; commented-out code. | Delete. |

**The provenance test.** Before marking a history comment DELETE, confirm the
story is actually recoverable — `git log --follow -p -- <file>`, `git blame`,
a referenced PR. If it is not (the decision lived only in a chat and the
comment is its sole record) **and it still matters**, the verdict is RELOCATE —
or record it in the pruning commit's message body, which turns the deletion
itself into the durable record.

Detailed signals, worked examples, and edge cases (TODOs, commented-out code,
doc comments consumed by tooling): see
[references/classification.md](references/classification.md).

Present the plan as a table — `file:line`, the annotation (truncated), verdict,
and *where the information lives afterwards* — then **stop and ask for
approval**. Do not edit anything in Phase 1.

## Phase 2 — Execute and verify

1. **Apply the approved verdicts.** Annotations only — never change code in the
   same commit. If pruning exposes a code smell, report it; do not fix it here.
2. **Annotation-only diff check.** Review `git diff` and confirm every hunk
   touches only comments and developer docs. Where comment syntax can reach
   build output (HTML `<!-- -->` in templates), prove the output is unchanged:
   build before and after into separate directories and diff them.
3. **Run the project's checks.** Consult the task manifest (`package.json`
   scripts, `Makefile`, etc.) and run format/lint/test. Do not proceed with
   failures.
4. **Commit.** The message summarizes what was pruned and names where each
   relocated fact now lives. Rationale that was deleted and is recorded nowhere
   else goes in the message body — the commit message is the relocation target
   of last resort.

## Report

- Counts per verdict and files touched.
- Where each RELOCATE landed.
- Confirmation that the diff was annotation-only, checks passed, and (where
  applicable) build output was byte-identical.

## Timing — when to run

- After a multi-iteration change settles — before opening or finalizing a PR —
  so review reads the code, not the story of writing it.
- When a touched file reads like a changelog.
- On explicit request.

Not after every commit, and not mid-implementation: pruning while the code is
still moving churns the very comments that are guiding the work.

## Never

- Delete information that would then survive nowhere — not in code, docs, or
  git history including the pruning commit's message.
- Mix behavior changes into a pruning commit.
- Prune end-user content, license headers, or machine-read directives.
- Skip the approval gate between Phase 1 and Phase 2.
- Report success while the diff touches non-annotation lines or checks fail.
