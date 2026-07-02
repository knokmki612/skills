---
name: design-review
description: Evaluate a design or refactoring decision problem-first, treating design principles and patterns (SOLID, DRY, GoF patterns, etc.) as after-the-fact tools rather than goals to reach. Use when reviewing a design, judging whether to apply or invoke a principle/pattern, deciding whether code "should" be refactored, or whenever a review risks forcing code into a pattern for its own sake. Guards against pattern-fitting that drifts from the actual problem.
license: CC-BY-4.0
---

# Design Review, Problem-First

## Why

The common failure in design review is not lack of knowledge about principles and
patterns — it is **reaching for them too early**. Once a principle's name is on the
table ("this violates SRP", "this isn't DRY", "we should use a Strategy here"), the
review quietly re-centers on *making the code match the principle* instead of
*solving a real problem*. The principle becomes the goal; the original need gets
bent to fit it. The result is abstraction nobody asked for, indirection that pays
for a flexibility that never arrives, and "cleaner" code that is harder to change.

Principles and patterns are compressed hindsight about problems that recur. They are
**tools for problems**, not standards code must satisfy. So this skill fixes the
*order of reasoning*: establish the problem first, and only then let a principle
earn its way in by solving that problem more cheaply than the alternatives.

## The rule

> **A principle or pattern may only be named once a concrete, real problem it solves
> has already been stated.** Naming it earlier is the bug this skill exists to catch.

If you notice yourself typing a principle's name before you have written down a
specific, observable problem — stop, delete it, and go back to Step 1.

## Procedure

Work the steps in order. Do not skip ahead to Step 4; that is the whole point.

### Step 1 — State the problem in plain phenomena

Describe what is actually wrong using concrete, observable facts about *this* code:
what changes, what breaks, what is hard to read, what took long to trace.

- **Banned in this step:** the name of any principle, pattern, or "smell"
  (SRP, DRY, coupling, God object, Strategy, …). If you can only express the
  concern as "it violates X", you have not found the problem yet — you have found a
  label. Translate it into the underlying phenomenon.
- Good: "Adding a new payment provider means editing five unrelated files and one
  of them is a 400-line `switch`." Bad: "It violates OCP."

If you cannot state a concrete phenomenon, there may be no problem. Go to Step 5.

### Step 2 — Test whether the problem is real

Separate **observed facts** from **hypothetical futures**.

- **Observed:** it already happened, or is happening now (a bug shipped, a change
  was painful, a test is flaky, a reviewer got lost). These count.
- **Hypothetical:** "what if we later need to…", "this won't scale if…", "a future
  developer might…". These are speculative until evidence exists.

Apply YAGNI to hypotheticals: a speculative problem does not justify design work
now. Demote it to a note ("if X actually happens, revisit") and set it aside. Only
carry **observed** problems into Step 3.

### Step 3 — Locate the pain, concretely

For each real problem, pin down *where it hurts* using measurable or demonstrable
symptoms, not verdicts:

| Symptom to look for | How to make it concrete |
| --- | --- |
| Change amplification | "One logical change edits N files / N call sites." |
| Duplication that drifts | "Same rule in K places; they have already diverged / caused a bug." |
| Coupling | "Changing A forces a change in B though they are unrelated." |
| Readability | "It took me / a reviewer this long to answer a basic question about it." |
| Testability | "To test X you must stand up Y and Z; there is no seam." |

State the pain as a symptom with evidence. "This is highly coupled" is a verdict;
"changing the tax rule requires touching the PDF renderer" is a symptom. Carry
symptoms, not verdicts, into Step 4.

### Step 4 — Only now, consider principles and patterns

With a real problem (Step 2) and its concrete pain (Step 3) fixed, look for a fix.

- Brainstorm candidate solutions. A principle or pattern may enter here as **one
  candidate**, named for the first time.
- **Evaluation axis:** not "does the code match this principle?" but **"does this
  solve the Step 3 pain more cheaply than the alternatives?"** — cost counted in
  added indirection, concepts a reader must hold, and future change effort.
- **Always float at least one simpler alternative** and compare against it
  explicitly. The bar to clear: a rename, a helper function, a comment, inlining,
  deleting code, or moving one thing — a pattern must beat the simplest option that
  addresses the same pain, not just be *applicable*.
- When you need to confirm what a principle or pattern *actually means* (definitions
  are easy to misremember and misapply), consult
  [references/principles.md](references/principles.md). Open it to **verify a
  definition you are already considering** — not to browse for a principle to apply.

### Step 5 — Conclude

State the decision and the reason, tied back to the Step 3 pain.

- **"Change nothing" is a valid, first-class conclusion.** No real problem (Step 2),
  or no candidate that beats the simpler alternative (Step 4), means the right output
  is to leave the code alone — and to say why, so the non-action is deliberate.
- If you do recommend a change, name the specific pain it removes and the simpler
  option it beat.

## Anti-pattern self-check

Before finalizing, scan your own reasoning for these tells that the order slipped:

- A principle/pattern name appears **before** a concrete problem is stated.
- The justification is "best practice" / "cleaner" / "more maintainable" with no
  named pain behind it.
- The driving problem is hypothetical ("we might need…") with no evidence.
- No simpler alternative was considered, or it was dismissed without comparison.
- The recommendation adds a layer, interface, or indirection whose flexibility is
  not exercised by any current requirement.
- The review restated a "smell" as if the smell itself were the problem.

Any hit → return to the step where the slip happened.

## Output format

```
Problem (plain phenomena):   <what is observably wrong — no principle names>
Real or hypothetical:        <observed evidence, or demoted via YAGNI>
Concrete pain:               <symptom + evidence, not a verdict>
Options considered:          <candidate(s), incl. the simplest alternative>
Decision:                    <chosen option OR "change nothing">
Why this over the simpler option: <cost/benefit tied to the concrete pain>
```

## Reference

[references/principles.md](references/principles.md) — pointers to trusted, primary
sources for principle/pattern **definitions**, with a note on when to consult each
and the point each is most often misread on. It is a lookup for Step 4, not a menu
to shop from. Comprehensiveness is delegated to those external sources; searching
the web for a *solution approach* is fine, but confirm *definitions* against the
primary sources listed there.
