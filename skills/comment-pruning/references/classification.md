# Classification rubric — signals, examples, edge cases

## The three layers a comment can legitimately occupy

A keepable annotation fits one of these layers; anything else is a candidate
for REWRITE, RELOCATE, or DELETE.

1. **Guard — always keep.** A point that would puzzle someone reading the
   implementation, or a DO-NOT / pitfall. Keep a rationale only when it is
   specific to this spot, cannot be read off types, signatures, error messages,
   nearby code, or language semantics, and removing it would invite a breaking
   "improvement". Keep these minimal, with a `NOTE:` prefix, so the reasoning
   stays traceable.
2. **Convention why — once.** A codebase-wide convention written once, at the
   enforcement point (root/main, or the relevant function). A leaf-level echo
   of it is a drift source: DELETE the echo, keep the origin.
3. **Single-line what — writer's discretion.** A heading line over a following
   non-trivial multi-line block is fine; a one-line paraphrase of a single
   self-named call is not.

Design provenance — who decided what, when, and why — belongs in ADR / PR /
commit messages, not in code, *except* when it doubles as a layer-1 guard.

## Signals of historical narration

Any of these marks a comment as history-shaped. It is not an automatic DELETE —
run the provenance test and check for a living fact inside — but it is the
trigger to classify deliberately:

- Past tense about the codebase itself: "was", "used to", "previously",
  "moved from", "renamed from", "以前は", "従来は", "〜に伴い", "〜を廃止".
- Issue/PR numbers or dates used as narrative anchors: "since #618", "after
  the 2025 redesign". (A PR number *inside a kept guard* as a pointer to
  deeper discussion is fine — a pointer is not narration.)
- Change-request framing: "per review feedback", "as discussed", "the owner
  decided", "エージェントの判断で".
- Before/after comparisons whose "before" no longer exists in the tree.
- Apologies and journals: "temporary workaround until…" (check whether "until"
  already happened), "keeping this for now".

## Worked examples

### DELETE — provenance duplicated by git

```js
// 2025-11: blogs[].needs was removed in #618 and replaced by contactTheme.
// We used to filter here, but that logic moved to lib/blogs.js in #642.
const themes = blog.contactTheme;
```

Both sentences retell what `git log -p` and the referenced PRs record. The
current code is self-describing. Delete both lines.

### REWRITE — a living fact wrapped in a story

```js
// After #631 moved blogs to microCMS we now cache for 1s in production
// because webhook deploys must always see fresh content.
duration: isProd ? "1s" : "1h",
```

The history ("after #631 moved…") is git's job; the constraint (webhook deploys
must see fresh content) is a layer-1 guard. Rewrite:

```js
// Webhook-triggered production deploys must fetch fresh content; dev caches
// long to spare the API.
duration: isProd ? "1s" : "1h",
```

### KEEP — a guard that cannot be read off the code

```js
// NOTE: this file must have only a default export. A named export makes
// Eleventy's data resolution wrap everything in an object and breaks
// pagination. Helpers live in lib/blogs.js.
```

Nothing in types or nearby code reveals this; removing it invites the exact
breaking "improvement" it warns against. Keep, `NOTE:`-prefixed.

### RELOCATE — design rationale stranded in a leaf file

```njk
{# The title of every page except / must start with the page name, not the
   company name, because search snippets truncate at ~30 chars. #}
```

This is a site-wide content convention, not a fact about this template. Move it
to the docs section that defines title rules (or create one), leave at most a
pointer if the spot is a genuine pitfall, and delete the rest.

## Edge cases

- **TODO / FIXME.** Keep if still actionable and accurate. If stale or big
  enough to deserve tracking, convert to an issue and delete the comment
  (the issue link goes in the pruning commit message).
- **Commented-out code.** DELETE, always — git preserves it. If it encodes an
  alternative worth remembering, describe the alternative in one guard line or
  in docs instead of keeping dead code.
- **Doc comments consumed by tooling** (JSDoc/TSDoc feeding editors or docs
  builds, docstrings feeding `help()`): they are API surface, not annotation
  sediment. Prune historical narration *inside* them, keep the structure.
- **Comments citing external URLs** (vendor docs, specs): keep when the URL is
  load-bearing for a guard; delete when it decorated a narration that is being
  deleted anyway.
- **Generated files**: never hand-edit; fix the generator or skip.
- **Config files with sparse comment support** (JSON via `.jsonc`, YAML): same
  rubric; be extra careful that the parser accepts the file after edits — run
  the project's checks as always.
- **A comment that is wrong.** Not a pruning question — a wrong guard is a bug.
  Flag it in the report; fixing it may need a code-understanding pass outside
  this skill's annotation-only guarantee.
- **Ambiguity between REWRITE and DELETE.** Ask: if a competent reader saw only
  the current code, would the residual fact save them real effort *at this
  spot*? Yes → REWRITE down to that fact. No → DELETE.
