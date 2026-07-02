---
name: wip-commit-organization
description: Reorganize a messy stack of work-in-progress commits into a clean, reviewable history without ever losing the working diff. Analyze the commits a branch adds over its base, propose a reconstruction plan (order, granularity, messages), get approval, then rebuild the history and mechanically verify the final tree is byte-for-byte identical. Use after piling up commits — literal `wip` commits, `fixup`, `.`, "address review", or any ad-hoc checkpoint — when squashing, splitting, reordering, or rewording branch history, or tidying commits before opening or updating a PR.
license: CC-BY-4.0
---

# WIP Commit Organization

## Why

Real work produces messy history: `wip`, `fixup`, `.`, "oops", "address review",
commits that touch three unrelated things, fixes that belong in an earlier commit.
The goal is a history where **each commit is one logical change** — reviewable and
revertable on its own — in an order that reads top to bottom.

Rewriting history is destructive. `rebase`, `reset`, and `commit --amend` move refs
and can silently drop hunks. The one guarantee worth protecting above all else is
that **the final working tree does not change** — you are re-cutting the *path* from
base to tip, never the *destination*. This skill enforces that with an explicit
backup and a mechanical end-state check, and it puts every irreversible operation
behind a single approval gate.

## Scope — what gets reorganized

"WIP commits" does **not** mean only commits literally titled `wip`. It means
**every commit the branch adds on top of its base branch** — the full ad-hoc stack.
The reorganization target is the range `<base>..HEAD`.

Determine `<base>` before anything else, and confirm it with the user if ambiguous:

```bash
git merge-base HEAD origin/main      # typical: fork point from the default branch
git log --oneline <base>..HEAD       # this exact set is what gets reorganized
```

- The base is the commit the branch diverged from — usually the merge-base with the
  default branch (`main` / `master`) or the PR's target branch. It is **not**
  reorganized; only commits *after* it are.
- If the branch tracks an upstream that already has your commits, or history was
  already pushed and shared, say so — rewriting shared history needs the user's
  explicit go-ahead (see [Already-pushed history](#already-pushed-history)).

## Execution model

Two phases separated by an approval gate. Never merge them.

```
Phase 1  ANALYZE  →  present reconstruction plan  →  ⛔ USER APPROVAL
Phase 2  EXECUTE  →  rebuild history  →  VERIFY end-state  →  report
```

The gate is the safety mechanism, not a formality. Do not run any
history-rewriting command in Phase 1, and do not start Phase 2 without approval.

## Phase 0 — Preconditions and backup

Run every step. Stop and surface the problem if any fails.

1. **Clean tree.** `git status --porcelain` must be empty. Stash or commit first —
   a dirty tree makes the end-state check meaningless.
2. **Record the original tip.**
   ```bash
   git rev-parse HEAD          # note this SHA in your working notes and the final report
   ```
3. **Create a backup branch** (first line of defense; reflog is only a fallback):
   ```bash
   git branch backup/wip-<timestamp>      # e.g. backup/wip-20260702-1530
   ```
   Use a timestamp you were given or can read from `git log -1 --format=%cd`; do not
   invent one. Confirm it exists with `git branch --list 'backup/wip-*'`.
4. **Capture the reference diff** so the end-state check has a target:
   ```bash
   git rev-parse backup/wip-<timestamp>   # === the original HEAD SHA from step 2
   ```

The working branch keeps its name throughout; the backup branch is what you restore
from if anything goes wrong.

## Phase 1 — Analyze and plan

Read the range and produce a reconstruction plan. This phase is **read-only** —
`git log`, `git show`, `git diff`, `git diff <base>..HEAD` only.

For large or noisy stacks you may delegate this reading to a read-only analysis
subagent (`context: fork`) to keep the main context clean — see
[Optional analysis subagent](#optional-analysis-subagent). The subagent only
*reads and proposes*; it never runs a rewriting command.

Apply the [granularity and ordering principles](#semantic-principles) below, then
present the plan in this fixed format:

```
BASE:          <base sha> (<branch/ref it comes from>)
ORIGINAL TIP:  <sha>   BACKUP: backup/wip-<timestamp>
ORIGINAL:      N commits  →  PROPOSED: M commits

Proposed commits (in final order):

1. <type>: <message>
   - includes: <files / hunks>
   - from original: <which original commits/hunks this pulls together>
   - why here: <dependency rationale — what it must come before/after>

2. ...

Notes:
- <squashes, splits, drops, or reorderings that need calling out>
- <anything that can't build/test standalone, if unavoidable>
```

Then **stop and ask for approval.** Do not proceed until the user accepts or edits
the plan.

## Phase 2 — Execute

Pick the technique per the plan. Reordering / squashing / rewording is
`rebase -i`; re-cutting granularity is `reset --soft` + re-staging.

### Reorder, squash, reword, drop

```bash
git rebase -i <base>
```

In the todo list: reorder lines to set commit order, `squash`/`fixup` to merge,
`reword` to fix messages, `drop` to delete, `edit` to pause and adjust a commit.

### Re-cut granularity (split one commit, redraw boundaries)

The main tool when the *hunk boundaries* are wrong, not just the order:

```bash
git reset --soft <base>     # uncommit everything, keep all changes staged — tree untouched
git reset                   # optional: move changes to unstaged so you can re-pick hunks
git add -p                  # stage hunk by hunk into one logical commit
git add -e                  # hand-edit the staged patch when -p's hunks are too coarse
git commit -m "..."         # repeat add-p / commit per planned commit
```

`reset --soft` is safe by construction: it moves only the branch ref, never the
files. The working tree is identical before and after — you are only re-deciding how
to package the same changes.

### Move a hunk into an earlier commit

`git rebase -i <base>`, mark the target commit `edit`, then `git add -p` the hunk
and `git commit --amend`, or use `git commit --fixup=<sha>` and finish with
`git rebase -i --autosquash <base>`.

### Keep each commit buildable (optional but recommended)

If the plan claims every commit builds/tests, enforce it:

```bash
git rebase -i --exec "<test or build cmd>" <base>
```

The rebase stops at the first commit that fails the command, so bisectability is
verified, not just asserted.

## Phase 2 (cont.) — Verify end-state  ⛔ completion gate

The single check that proves nothing was lost:

```bash
git diff backup/wip-<timestamp> HEAD --stat    # MUST be empty
git diff backup/wip-<timestamp> HEAD           # MUST print nothing
```

- **Empty diff** → the reorganized history reproduces the original final tree
  exactly, byte for byte. Success.
- **Non-empty diff** → the rewrite changed the result. **Do not present this as
  done.** Restore and report:
  ```bash
  git reset --hard backup/wip-<timestamp>      # back to the original tip
  ```
  Then explain what diverged and either retry the plan or hand back to the user.

Also sanity-check the shape:

```bash
git log --oneline <base>..HEAD                 # matches the approved plan
git rebase -i --exec "<test cmd>" <base>        # if buildability was promised
```

## Report

Close with:

- Original tip SHA and the backup branch name (so the user can restore anytime).
- Before/after commit counts and the final `git log --oneline <base>..HEAD`.
- Confirmation that `git diff backup/... HEAD` was empty.
- How to discard the backup once satisfied: `git branch -D backup/wip-<timestamp>`.

Do **not** delete the backup branch yourself — leave it for the user.

## Semantic principles

**Granularity — one commit, one logical change:**

- Each commit stands alone: it can be described in one line and would make sense as a
  single `revert`.
- Do not mix refactor with feature change. Split pure moves, renames, and
  formatting into their own commits.
- Tests go with the implementation they cover, in the same commit or immediately
  after.

**Order — dependencies flow downstream to upstream:**

- Foundations first: type definitions, schemas, utilities, config — then the code
  that uses them.
- Prefer an order where **every commit builds and its tests pass**, so the history
  stays `git bisect`-able.
- Group related changes adjacently; don't interleave unrelated threads.

## Already-pushed history

If `<base>..HEAD` has already been pushed (especially to a shared branch or an open
PR):

- Rewriting means the remote branch will need a **force push**
  (`git push --force-with-lease`). Confirm with the user before doing it.
- If others may have based work on these commits, rewriting can disrupt them — flag
  this explicitly rather than proceeding.
- The backup branch and end-state check still apply unchanged.

## Optional analysis subagent

When the stack is large enough that reading every diff would pollute the main
context, delegate **Phase 1 only** to a read-only subagent (`context: fork`):

- Input: the base ref and `<base>..HEAD`.
- The subagent runs read-only git commands and returns the reconstruction plan in
  the exact format above.
- It must **not** run `rebase`, `reset`, `commit`, or any ref-moving command.

Keep the backup, approval gate, execution, and end-state verification in the main
skill flow — never behind the subagent, where a failure would be invisible and
recovery couldn't be driven. Calling the subagent is optional; the skill is complete
without it.

## Edge cases

See [references/edge-cases.md](references/edge-cases.md) when the range isn't a clean
linear stack or the rewrite hits trouble:

- Merge commits inside `<base>..HEAD` (flatten vs. preserve)
- Conflicts during a reorder/split rebase
- Commits that can't build standalone
- Empty commits left after squashing
- Binary files, generated artifacts, submodule pointer bumps
- A single-commit range, or an ambiguous base

## Never

- Rewrite history without a backup branch and a recorded original SHA.
- Skip the approval gate between Phase 1 and Phase 2.
- Report success while `git diff backup/... HEAD` is non-empty.
- Delegate a history-rewriting command to a subagent.
- Force-push a shared branch without the user's explicit confirmation.
