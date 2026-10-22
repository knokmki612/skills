---
name: adversarial-verification
description: Verify a claim about code by trying to break it, not by re-reading the reasoning that produced it — a fix resolves the bug, this is the root cause, the refactor preserves behavior, the change touches nothing else, the test covers it, a review finding is real. Pin the claim as a falsifiable statement, pick attacks from a catalog (revert the fix and watch the test fail, boundary and hostile inputs, every caller, error paths, environment differences, before/after diffs), actually run them, and report CONFIRMED only when every attack failed with recorded evidence; REFUTED with a counterexample; UNRESOLVED for anything not run. Use before reporting a change as fixed, done, verified, or safe, when a root-cause hypothesis forms, when tests pass but the change was never exercised directly, when checking a finding from a review, or when asked to verify, double-check, confirm, or make sure.
license: CC-BY-4.0
---

# Adversarial Verification of Code

## Why

Handed a claim about code — "the fix works", "this is the root cause", "the
refactor changes nothing" — an agent tends to **confirm** it: re-read the diff,
re-run the test that was already green, restate the argument that produced the
claim, and report success. That loop never leaves the evidence the claim was
built from, so it cannot find what the claim missed. The failures that reach
users live exactly there: the input the fix never saw, the second caller of the
changed function, the test that passes with the fix reverted because it never
exercised the change.

Verification is a different activity from confirmation. It starts from "how
would this be false?", builds concrete attempts to make it false, runs them,
and earns a verdict from their outcome. A claim that survives a serious attempt
to break it is worth something; a claim that was merely re-derived is not.

## The rule

> **A claim about code is unverified until a deliberate attempt to falsify it
> has been made, run, and recorded.** Reproducing the claim's own evidence is
> confirmation, not verification. The burden of proof sits on the claim; the
> verifier's job is to produce a counterexample, and only failing that, to say
> so with the attempts listed.

Two corollaries:

- **Do not carry the original reasoning in.** Whether you made the claim
  yourself or received it, start from the claim and the code, not from the
  argument for it. The argument tells you where the author already looked;
  the counterexample is somewhere else.
- **A verification with zero attacks run is invalid.** "I read it carefully and
  it looks right" is a prediction, not a result.

## Scope

A claim is in scope when it is a statement about code whose truth can be
tested by running, diffing, or tracing that code:

- A change fixes a reported bug.
- A located line, condition, or interaction is the root cause of a symptom.
- A refactoring preserves observable behavior.
- A change has no effect outside its intended surface.
- A test covers the behavior it is named for.
- A review finding (bug, regression, missed case) is a real defect.
- A measured property (faster, fewer allocations, no longer flaky) holds.

Out of scope: judgments with no runnable test — whether a design is good,
whether a name is clear, whether prose is accurate. Those take a different
kind of review.

## Procedure

Work the steps in order. Step 3 is the one that gets skipped; it is the only
one that produces evidence.

### Step 1 — Pin the claim as a falsifiable statement

Rewrite the claim so that a specific observation would refute it. State:

- **Input / state:** what is fed in, what the world looks like first.
- **Action:** what runs.
- **Expected observable:** output, exit code, file state, log line, timing.
- **Scope:** which callers, environments, and inputs the claim covers. "Works"
  with no scope is not a claim yet.

Then write the refutation sentence explicitly: *"This is false if ⟨input⟩
under ⟨conditions⟩ produces ⟨observation⟩."* If you cannot write it, the claim
is too vague to verify — split it, or narrow it until you can.

Compound claims ("fixed the bug and didn't break anything") split into one
line each. Each line gets its own attacks and its own verdict.

### Step 2 — Build the attack list

For each claim, choose attacks from
[references/attack-catalog.md](references/attack-catalog.md). Pick by asking
**"if this claim is wrong, where is it most likely wrong?"** — the answer is
almost always a path the original work did not exercise: an input shape it
never saw, a caller it did not grep for, a branch its test does not reach.

Requirements for the list:

- **At least three distinct attacks per claim**, from different categories of
  the catalog. One attack is a spot check.
- **Each attack is concrete before it runs:** the exact command or input, and
  the observation that would count as the claim breaking. An attack whose
  failure signature you cannot name is not ready.
- **The revert check is mandatory whenever a test is offered as evidence:**
  undo the change, run the test, and it must fail. A test that passes either
  way is not evidence for the claim, whatever else it checks.
- **Every caller of a changed symbol is in the list** for claims of the form
  "no effect elsewhere" or "preserves behavior". Find them with search, not
  memory, and include dynamic ones (string lookups, reflection, config-driven
  dispatch).

Write the list down before running anything. It is the record of what the
verdict rests on, and it stops the list from shrinking to whatever happened to
pass.

### Step 3 — Run the attacks

Execute every attack on the list. For each, record:

```
Attack:     <name from the catalog, specialised to this claim>
Command:    <exact invocation, or the input and how it was applied>
Expected if false: <the failure signature from Step 2>
Observed:   <actual output, exit code, diff, timing — pasted, not paraphrased>
Result:     HELD | BROKE | NOT RUN (<why>)
Evidence:   checked: <path:line> / <command> | inferred
```

Rules while running:

- **Run it.** Reading the diff and predicting the outcome is `inferred` and
  counts as NOT RUN. Only an executed command, an actual diff, or a traced
  path with `path:line` counts as `checked`.
- **Do not adjust the claim mid-run to fit what you see.** If an attack breaks
  the claim as stated, that is a BROKE. Narrowing the claim is done in Step 4,
  in the open.
- **Do not fix anything yet.** A verifier who starts patching has switched
  roles and will confirm the patch next. Finish the attacks, report, then fix
  as a separate step with its own verification.
- **Do not stop at the first HELD.** Attacks that pass are the expected case;
  the list exists to find the one that does not.
- **A NOT RUN is a NOT RUN.** Could not reproduce the environment, no fixture
  for that input, the dependency is not installed: say so. Do not quietly
  replace the attack with an easier one.

### Step 4 — Verdict

Assign exactly one verdict per claim:

| Verdict | Requires |
| --- | --- |
| **CONFIRMED** | Every listed attack ran and HELD, including the revert check if a test was cited. |
| **REFUTED** | At least one attack BROKE, with a reproduction someone else can run. |
| **UNRESOLVED** | No attack BROKE, but at least one is NOT RUN. Name each one and why. |

Verdict rules:

- **UNRESOLVED never rounds up to CONFIRMED.** "Probably fine" is a prediction;
  the verdict reports what was tested.
- **Narrowing is allowed but visible.** If the attacks confirm a smaller claim
  than the one made ("fixed for ASCII paths; untested for non-UTF-8"), issue
  CONFIRMED for the narrowed claim and state the original claim's residue as
  UNRESOLVED or REFUTED. Never silently report the narrow result under the
  broad claim.
- **REFUTED comes with a counterexample, not an opinion.** The command, the
  input, the observed output. If it cannot be reproduced on demand, it is a
  suspicion to pursue, not a refutation.
- A REFUTED root-cause claim should say which it is: the symptom persists with
  the cause removed, or the symptom has another source as well.

### Step 5 — Report

Use the output format below. The report is for a reader who did not watch the
run: they must be able to re-execute any attack from what is written.

## Anti-pattern self-check

Before finalizing, scan the verification for these tells. Each is listed under
the step it slipped at; any hit → return to that step.

**Step 1 — the claim**

- The claim has no refutation sentence, or the sentence is "it would be false
  if it didn't work".
- A compound claim got one verdict.

**Step 2 — the attacks**

- Every attack exercises the same path the original work already exercised.
- A test is cited as evidence and the revert check is not on the list.
- Callers were listed from memory, or the search covered only the obvious
  literal name.
- An attack has no named failure signature.

**Step 3 — the run**

- An `inferred` entry reads as if it had been executed.
- The original fix was modified during the run, and the modified version is
  what the attacks were run against.
- An attack on the list is missing from the results, with no NOT RUN entry.
- "Tests pass" appears without the command that ran them and which tests they
  were.

**Step 4 — the verdict**

- CONFIRMED with a NOT RUN in the results.
- The verdict covers a narrower claim than Step 1 stated, and the narrowing is
  not written down.
- REFUTED without a reproduction, or a BROKE explained away as "that input
  would never happen" without evidence that it cannot.

## Output format

```
Claim (as stated):          <the original claim, verbatim>
Claim (falsifiable form):   <input / action / expected observable / scope>
False if:                   <the refutation sentence>

Attacks:
  1. <name>  — <command or input>
     expected if false: <signature>
     observed:          <pasted output / diff / path:line>
     result:            HELD | BROKE | NOT RUN (<why>)   [checked | inferred]
  2. ...
  (revert check present when a test is cited)

Verdict:                    CONFIRMED | REFUTED | UNRESOLVED
  scope confirmed:          <what the attacks actually cover, if narrower>
  counterexample:           <reproduction, if REFUTED>
  not run:                  <attacks and reasons, if UNRESOLVED>
```

## Reference

[references/attack-catalog.md](references/attack-catalog.md) — the catalog of
attacks grouped by what they target (the test as evidence, inputs, callers,
state and environment, error paths, root-cause claims, behavior preservation,
measured properties), each with its failure signature and the claim types it
applies to. Open it in Step 2 to build the list; it is a source of attacks to
run, not a checklist to tick.
