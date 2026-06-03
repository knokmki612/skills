# Edge Cases

Consult this file when the deletion target is not a plain tracked / untracked / ignored path in a normal working tree.

## Submodules

Removing a submodule requires deinit before `git rm`, otherwise stale state lingers under `.git/modules/`:

```bash
git submodule deinit -f -- <submodule-path>
git rm -- <submodule-path>
rm -rf .git/modules/<submodule-name>   # only this internal git path; user-tree rm rules still apply elsewhere
```

The `rm -rf .git/modules/<submodule-name>` step is the one place `rm` is acceptable: it targets git's internal storage, which git itself does not clean up after `deinit`. Confirm the path with the user before running.

## Nested git repositories (non-submodule)

`git clean` refuses to descend into a directory that is itself a git repo unless `-f` is doubled (`-ff`). Treat this as a signal: the nested repo is probably someone's work-in-progress.

1. Confirm with the user that the nested repo is disposable.
2. Inside the nested repo, run the normal workflow first (`git status`, `git rm`, etc.) to preserve its history if relevant.
3. Only then use `git clean -ffd -- <nested-path>` from the outer repo.

## Bare repositories

This skill does not apply. Bare repos have no working tree — there is nothing to "delete" in the file-on-disk sense. Refuse and surface to the user.

## `.git/` and files under it

Never use this skill — or `rm` — to modify anything under `.git/` directly. Use porcelain commands:

- Untrack a file but keep on disk: `git rm --cached -- <path>`
- Remove a remote: `git remote remove <name>`
- Delete a branch: `git branch -D <name>`
- Wipe a worktree: `git worktree remove <path>`

The only exception is `.git/modules/<submodule-name>` after submodule deinit (see above).

## Worktrees

Linked worktrees (`git worktree add ...`) must be removed via `git worktree remove <path>`, not `git clean` or `rm`. That command both deletes the working files and prunes `.git/worktrees/<name>`.

If `git worktree remove` refuses (dirty state), inspect inside the worktree first and run the normal workflow there, then retry.

## Ignored content the user wants kept

A path can be both untracked and ignored. `git clean -fx` will remove both classes — that may delete more than the user intends (e.g. `node_modules/`, `.env`, build caches).

- Default to **named paths only**: `git clean -fx -- <specific/path>` rather than a directory.
- When deleting a directory subtree, dry-run with `-ndx` and read the full list to the user before executing.
- If the user only wants the untracked-but-not-ignored subset, drop `-x`/`-X`.

## Symlinks

`git rm` and `git clean` operate on the link itself, never on the link target. That is the desired behaviour — do not resolve the symlink first.

## Sparse-checkout

Files outside the current sparse cone are not on disk, so `rm` would be a no-op anyway. To stop tracking them:

```bash
git sparse-checkout set <remaining-paths>     # adjust the cone
git rm --cached -- <path>                     # if also untracking
```

Do not try to delete sparse-excluded paths with `git clean` — they are not present locally to begin with.

## LFS-tracked blobs

`git rm -- <path>` works normally and removes the pointer. The actual LFS object remains in `.git/lfs/objects/` and on the remote until pruned (`git lfs prune`). That is a separate concern; do not invent a workflow to wipe LFS storage as part of a delete request unless the user asks.

## Read-only / permission-denied files

If `git rm` fails on a read-only file, fix the permission with `chmod` first. Do not bypass with `rm -f` outside git.

## Empty directories

Git does not track empty directories. An empty directory is therefore always untracked. Use `git clean -fd -- <path>` rather than `rmdir`. This keeps the deletion behaviour uniform with the rest of the skill and still passes through git's safety flags.
