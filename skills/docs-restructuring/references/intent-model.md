# Intent model — reader questions, defect catalog, target patterns

## The four reader questions

A developer opens documentation with exactly one question. Structure serves
them when each section answers one question and its heading says which.

| Question | Content | Typical form |
| --- | --- | --- |
| **Understand** — how does this work, why is it built this way? | Concepts, architecture, design rationale | Prose, diagrams |
| **Not break** — what must I not do? | Invariants, contracts, pitfalls | Short imperative statements, each with a pointer to the check that enforces it |
| **Do** — how do I accomplish X? | Procedures: setup, preview, deploy, add-a-page | Numbered steps, copy-pasteable commands |
| **Look up** — what is the value of Y? | Reference data: vocabularies, env vars, state matrices | Tables |

This is the Diátaxis framework's split (explanation / how-to / reference,
with tutorials as guided *do*), plus **not break** as its own class: in
developer docs, invariants are the highest-stakes content and deserve a home
of their own instead of being buried inside explanations.

Mixing intents is the failure that makes readers read everything to find
anything: scanning cost grows with the number of undifferentiated options,
so a list that interleaves a concept, two invariants, a procedure, and a
vendor constraint must be scanned in full for any of them.

## Defect catalog

All examples are fictional.

- **Mis-leveled hierarchy.** An `#### Architecture` subtree of 150 lines
  filed under `## Local development` because that is where it was first
  written. The heading tree now lies; navigation fails; readers fall back to
  linear reading. Fix: promote the subtree to its own top-level section.
- **Intent mixing.** One bullet list under "Articles" containing: how the two
  content types differ (understand), "never sort in templates — the ordering
  lives in one function" (not break), "run `clean:cache` to see CMS edits
  locally" (do), and the CMS plan's API-key limitation (understand,
  vendor-specific). Fix: split the list into its intent homes; each bullet
  moves verbatim.
- **Duplicated truth.** The doc enumerates the seven values of a form's
  option list, which also exist in the template that renders them — two
  sources, one future drift. Same when a doc restates a rule a test enforces.
  Fix: state the vocabulary/rule at its enforcement point only; the doc keeps
  one line: what is constrained, and which file or check is canonical.
- **Monolith.** A single README carrying setup, architecture, content rules,
  and operations past the point of navigability. Fix: split into `docs/` by
  reader question, with the README reduced to an entry map (what this is,
  quickstart, where everything else lives).
- **Systemically orphaned conventions.** Site-wide rules scattered across
  the leaf files where each was first needed, with no conventions home. (A
  *single* stranded passage is comment-pruning's RELOCATE; a pattern of them
  means the structure lacks a home — create it here.)
- **Buried lede.** A critical invariant sitting mid-list between a caching
  tip and a vendor note. Fix: invariants first, or in their own subsection —
  position should encode importance.

## Target patterns

- **README as entry point**: what this is (2–3 sentences) → quickstart →
  a map of where everything else lives. Depth belongs in `docs/` (or clearly
  intent-named sections), not stacked in the entry point.
- **Intent-pure sections**, heading depth ≤ 3, and ~5–7 siblings per level —
  beyond that, regroup rather than append. Order sections by frequency of
  need; put *not break* where it cannot be missed.
- **One canonical home per fact.** Precedence: code → enforced check (test,
  schema, lint) → exactly one doc section. Docs point down this chain, never
  sideways at each other's restatements.
- **Form follows content type**: state combinations → table; relations and
  flows → diagram (e.g. mermaid) *with* a prose summary — the pairing serves
  both text-first and diagram-first readers, and diagrams are opaque to
  screen readers without it; procedures → numbered steps with commands in
  code blocks; invariants → one imperative sentence each plus the enforcing
  check.
- **Sentence form is inclusive form.** Short sentences, one clause each,
  keyword first; no stacking of bold, parentheses, and inline code inside one
  long sentence; consistent terminology (one name per concept, everywhere).
  This measurably helps dyslexic readers and costs fluent readers nothing.
  Per-unit enforcement of this belongs to `comment-pruning`; here it shapes
  what the target structure asks each section to hold.
