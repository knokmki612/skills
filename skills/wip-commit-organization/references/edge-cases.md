# Edge cases

Load these when the range `<base>..HEAD` isn't a clean linear stack, or the rewrite
hits trouble mid-flight. The backup branch and the empty-`git diff backup HEAD`
end-state check from `SKILL.md` still apply to every case below — they are the
invariant, not the exception.

## Merge commits inside the range

If `git log --oneline <base>..HEAD` shows merges (a merged-in branch, or the base
branch merged back in), a plain `git rebase -i <base>` will flatten or refuse them.

- Decide with the user whether the intended shape is **linear** (drop the merges,
  replay the work as a flat stack) or **preserved**.
- Linear is usually what "tidy the WIP stack" means. Prefer rebasing onto an updated
  base instead of carrying back-merges:
  ```bash
  git rebase --onto <new-base> <old-base>      # replay only your commits, linearly
  ```
- To keep merges, use `git rebase -i --rebase-merges <base>` and edit the generated
  `label`/`merge` todo lines carefully.
- The end-state check is unaffected: whatever the topology, `git diff backup HEAD`
  must still be empty.

## Conflicts during rebase

A rewrite that reorders or splits commits can conflict even though the final tree is
unchanged — intermediate states differ.

- Resolve each conflict, `git add` the result, `git rebase --continue`.
- If a resolution goes wrong, `git rebase --abort` returns you to the pre-rebase tip
  (still backed up either way).
- After finishing, the empty `git diff backup HEAD` is what proves the conflict
  resolutions didn't alter the destination. If it's non-empty, a resolution was
  wrong — reset to the backup and redo.

## Commits that can't build standalone

Sometimes a clean split leaves an intermediate commit that won't compile (e.g. a
symbol is used before the commit that introduces it, because the plan split them).

- Prefer reordering so the dependency lands first. If genuinely impossible without
  merging the two changes, say so in the plan's Notes and squash them into one
  commit rather than shipping a broken intermediate.
- `git rebase -i --exec "<build>" <base>` surfaces exactly which commit breaks.

## Empty commits after reorganization

Squashing or moving hunks can leave a commit with no net change.

- `git rebase -i` drops empties by default; add `--empty=drop` to be explicit, or
  `--keep-empty` if an intentional empty marker must survive.

## Binary files and generated artifacts

`git add -p` cannot split a binary hunk. Stage binaries whole with `git add -- <path>`
and assign each to a single planned commit. The end-state `git diff` still validates
them byte-for-byte.

## Submodules

Submodule pointer bumps are single-line changes to the gitlink — treat each as an
indivisible hunk and place it in the commit that logically owns the submodule update.
Do not try to `add -p` a gitlink.

## The range is a single commit

Nothing to reorder; the only meaningful operation is a split (`reset --soft <base>`
+ `add -p`) or a reword (`git commit --amend`). Skip the rebase machinery entirely.

## Base is ambiguous or the branch has no clear fork point

If `git merge-base` disagrees with what the user considers "the base" (rebased
branch, multiple upstreams, orphan history), stop and confirm the exact base ref
before touching anything. Reorganizing against the wrong base silently reshapes
commits the user considers finished.
