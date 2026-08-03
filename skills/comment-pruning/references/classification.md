# Classification rubric — signals, budgets, examples, edge cases

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
- Issue/PR numbers or dates used as narrative anchors: "since #123", "after
  the redesign". (A PR number *inside a kept guard* as a pointer to deeper
  discussion is fine — a pointer is not narration.)
- Change-request framing: "per review feedback", "as discussed", "the owner
  decided", "エージェントの判断で".
- Before/after comparisons whose "before" no longer exists in the tree.
- Apologies and journals: "temporary workaround until…" (check whether "until"
  already happened), "keeping this for now".

## Signals of drift

In practice, drifted references are often the *dominant* defect — more common
than history narration. A drifted reference is factually wrong but usually
fixable as an annotation-only REWRITE: update the pointer.

- References to files or symbols that moved or were renamed. Verify every
  "same rules as X", "keep in sync with X", "see X" — does X still exist, and
  does it still hold that logic?
- **Volatile anchors**: commit hashes, line numbers, timestamps, file paths
  deep in refactor-prone areas. They rot silently. If the fact matters,
  REWRITE it onto a stable anchor — a function name, a heading, a test ID.
  If it doesn't, DELETE.
- Cross-file duplication claims ("mirrors the logic in X") where one side has
  since changed: fix the pointer *and* flag the divergence in the report —
  whether the code disagreement is a bug is outside annotation-only scope.

## Load budgets — when every fact is justified but the whole is unreadable

Each annotation can pass the keep-test individually while the aggregate
overwhelms the reader. Humans hold roughly four chunks in working memory and
scan undifferentiated lists linearly, so budgets apply **per reading unit**,
not only per fact:

- **One annotation = one concern.** A block carrying several independent facts
  gets split or trimmed even if every fact is keepable.
- **Comment blocks ≤ ~4 lines**, except a guard that genuinely needs more.
- **Doc lists ≤ ~5 sibling bullets.** Beyond that, regroup by reader intent —
  *understand* (concept), *not break* (invariants), *do* (procedure), *look
  up* (reference). Mixing intents in one list forces a linear scan of
  everything to find anything.
- **One sentence, one clause**; put the keyword first. Avoid stacking bold,
  parentheses, and inline code inside a single long sentence — dense compound
  sentences tax every reader and especially dyslexic readers.
- **Facts enforced elsewhere get a pointer, not a restatement.** If a test,
  schema, or lint rule already guarantees an invariant, the doc says what is
  forbidden and *which check enforces it* — one line. The enforcement point is
  the canonical text; restating it creates the drift this rubric exists to
  remove.

Under these budgets REWRITE is mostly a *reshaping* operation — split,
regroup, pointer-ize, shorten. Deletion is its limiting case.

## Worked examples

All examples are fictional.

### DELETE — provenance duplicated by git

```js
// 2025-11: the `tags` field was removed in #412 and replaced by `topics`.
// We used to filter here, but that logic moved to lib/query.js in #430.
const topics = post.topics;
```

Both sentences retell what `git log -p` and the referenced PRs record. The
current code is self-describing. Delete both lines.

### REWRITE — a living fact wrapped in a story

```js
// After #310 moved fetching into the build step we now cache for 1s in
// production because webhook-triggered deploys must always see fresh content.
duration: isProd ? "1s" : "1h",
```

The history ("after #310 moved…") is git's job; the constraint (deploys must
see fresh content) is a layer-1 guard. Rewrite:

```js
// Webhook-triggered production deploys must fetch fresh content; dev caches
// long to spare the API.
duration: isProd ? "1s" : "1h",
```

### REWRITE — a drifted reference

```js
// Must stay in sync with the allowlist in src/data/render.js.
```

`render.js` was since split and the allowlist now lives elsewhere. Fix the
pointer, onto a stable anchor:

```js
// Must stay in sync with ALLOWED_HOSTS in lib/sanitize.js.
```

### REWRITE — an overloaded header (load budget)

```js
// Fetches the article list from the CMS at build time.
// - Each item is normalized by lib/articles.js; raw responses never reach
//   templates. Schema: tests/schemas/articles.ts.
// - Contract: publishedAt must be valid ISO; violations warn and are excluded.
// - Draft status cannot be read from the public list API; use the management
//   API instead. Not handled here.
// - Fetch failures throw and fail the build; zero items builds an empty list.
// - The API key lives in src/_data/env.js, overridable by env vars.
```

Six concerns in one block. Keep what a reader of *this file* needs (its role
and the behavior it enforces here); point to the enforcement points for the
rest:

```js
// Build-time fetch of the article list — external boundary only. Contract
// checks and normalization: lib/articles.js (schema: tests/schemas/articles.ts;
// behavior: README §Articles). Failures throw and fail the build; zero items
// is a valid empty list.
```

### KEEP — a guard that cannot be read off the code

```js
// NOTE: re-read the token file on every retry — the daemon rotates it
// mid-run, and caching it reproduces a 401 that only appears after ~1h.
```

Nothing in types or nearby code reveals this; removing it invites the exact
breaking "improvement" it warns against. Keep, `NOTE:`-prefixed.

### RELOCATE — design rationale stranded in a leaf file

```njk
{# Card headings must stay under 40 characters — the grid clips overflow
   without an ellipsis. #}
```

This is a site-wide content convention, not a fact about this one template.
Move it to the docs section that defines content rules (or create one), leave
at most a pointer if the spot is a genuine pitfall, and delete the rest.

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
- **A comment that is wrong about behavior.** A drifted *pointer* is a
  REWRITE (fix the reference). But a comment asserting something false about
  what the code *does* is a bug, not a pruning question — flag it in the
  report; fixing it may need a code-understanding pass outside this skill's
  annotation-only guarantee.
- **Ambiguity between REWRITE and DELETE.** Ask: if a competent reader saw only
  the current code, would the residual fact save them real effort *at this
  spot*? Yes → REWRITE down to that fact. No → DELETE.
