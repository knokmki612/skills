---
name: safety-deletion
description: Safely delete files and directories inside a git-managed working tree by routing every deletion through `git rm` or `git clean` instead of destructive commands (`rm`, `rmdir`, `shred`, `unlink`, `find -delete`, `truncate`). Use whenever removing any path from a repository — source files, generated artifacts, stray downloads, ignored caches, empty directories — so deletions remain recoverable through git history, the reflog, or at minimum go through git's working-tree safety checks. Trigger before invoking any non-git deletion command and whenever the user asks to delete, remove, drop, wipe, purge, or clean up files.
---

# Safe Deletion via Git

## Why

Plain `rm` / `rmdir` / `shred` bypass git entirely and are unrecoverable. Routing deletions through git keeps them reversible:

- **Tracked** paths land in history — `git reflog` and `git checkout` can restore them.
- **Untracked / ignored** paths still pass through `git clean`'s explicit-intent flags (`-f`, `-d`, `-x`, `-X`), which fails closed instead of silently nuking unrelated state.

## Workflow

1. **Confirm a git working tree.** Run `git rev-parse --is-inside-work-tree`. If it prints anything other than `true`, jump to [Outside a git repository](#outside-a-git-repository).
2. **Classify each target.** Use the table below to decide which bucket every path belongs to.
3. **Dry-run before any `git clean`.** Replace `-f` with `-n` first and inspect output.
4. **Execute** the matching command.
5. **Verify** with `git status` afterwards. No leftover surprises.

## Classifying a path

| Probe | Meaning |
| --- | --- |
| `git ls-files --error-unmatch -- <path>` exits 0 | Tracked (committed or staged) |
| `git status --short -- <path>` shows `??` | Untracked |
| `git check-ignore -v -- <path>` matches | Ignored |

A path can span multiple buckets across its subtree (e.g. directory contains tracked + ignored). Treat each subset with its matching command.

## Commands

Always place `--` before paths so they aren't parsed as flags.

| Target state | Command |
| --- | --- |
| Tracked file | `git rm -- <path>` |
| Tracked directory (recursive) | `git rm -r -- <path>` |
| Tracked **and** locally modified — loss of edits is intentional | `git rm -f -- <path>` |
| Newly `git add`-ed, want to undo and delete on disk | `git rm -f -- <path>` |
| Newly `git add`-ed, want to unstage but keep on disk | `git rm --cached -- <path>` |
| Untracked file | `git clean -f -- <path>` |
| Untracked directory | `git clean -fd -- <path>` |
| Ignored only | `git clean -fX -- <path>` |
| Untracked + ignored together | `git clean -fdx -- <path>` |

For any `git clean` invocation, run it once with `-n` substituted for `-f` first:

```bash
git clean -ndx -- <path>...   # preview
git clean -fdx -- <path>...   # execute
```

After `git rm`, finish the deletion with a commit (or instruct the user to). Until commit, the removal lives only in the index, but `git restore --staged --worktree -- <path>` still recovers it.

## Outside a git repository

When `git rev-parse --is-inside-work-tree` does not print `true`:

1. Do **not** silently fall back to `rm`. There is no recovery net.
2. Surface the situation and offer one of:
   - `git init` the directory and commit a baseline first, then proceed normally.
   - Use a trash CLI when available: `trash-put`, `gio trash`, `trash` (macOS Homebrew).
   - Proceed with `rm` only after the user explicitly confirms with full awareness.

## Prohibited commands

Never issue these against the working tree as a substitute for the above:

- `rm`, `rm -rf`
- `rmdir`
- `shred`
- `unlink`
- `find ... -delete`, `find ... -exec rm ...`
- `truncate -s 0` against existing files to "blank them out"

If a build script, generator, or tool insists on calling one of these, raise it with the user rather than running it silently.

## Edge cases

See [references/edge-cases.md](references/edge-cases.md) when the target involves:

- Submodules
- Nested git repositories or worktrees
- Bare repositories
- `.git/` itself or files under it
- Files matched by `.gitignore` that the user wants to keep on disk
- Symlinks, sparse-checkout cones, LFS-tracked blobs
