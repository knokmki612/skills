# Attack catalog

Attacks are grouped by what they target. Each entry names the claim types it
applies to, how to run it, and the **failure signature** — the observation that
means the claim broke. Build the Step 2 list from here: pick by where the claim
is most likely wrong, take at least three from different groups, and name the
signature before running.

Claim type shorthand used below:

| Code | Claim |
| --- | --- |
| FIX | A change fixes a reported bug |
| CAUSE | A located line / condition / interaction is the root cause |
| SAME | A refactoring preserves observable behavior |
| LOCAL | A change has no effect outside its intended surface |
| TEST | A test covers the behavior it is named for |
| FIND | A review finding is a real defect |
| MEAS | A measured property (faster, smaller, not flaky) holds |

## A. The test as evidence

Whenever a passing test is offered as evidence, attack the test before trusting
it. A test proves nothing about a change it does not exercise.

### A1. Revert check — *mandatory when a test is cited* (FIX, TEST, FIND)

Undo the change (`git stash`, `git revert --no-commit`, or edit it back), run
the same test, restore the change.

- **Failure signature:** the test still passes without the change. The test is
  not evidence for this claim; find or write one that fails on revert before
  proceeding.

### A2. Reach check (TEST, FIX)

Confirm the test actually executes the changed lines: coverage output, a
deliberate `raise`/`panic`/`throw` inserted at the changed line (then removed),
or a print of a sentinel.

- **Failure signature:** the inserted failure is not hit; coverage shows the
  lines unexecuted. The changed code is mocked, skipped, or behind a branch the
  test does not take.

### A3. Mock audit (TEST, FIX, SAME)

List every mock, stub, fake, and fixture the test uses. For each, ask whether
the thing being verified is on the other side of it.

- **Failure signature:** the behavior under claim lives inside a mocked
  dependency, or the fixture encodes the pre-fix wrong output as expected.

### A4. Assertion audit (TEST)

Read what the test asserts, not what it is named. Weaken the implementation
on purpose (return a constant, drop a branch) and run the test.

- **Failure signature:** the test still passes with the implementation broken.
  Its assertions do not pin the behavior its name promises.

### A5. Which tests ran (FIX, LOCAL, SAME)

Record the exact command and the selected set. Check for skip markers,
`only`/`focus`, excluded directories, and whether the suite that covers the
changed module was in the run at all.

- **Failure signature:** the relevant suite was skipped, filtered out, or the
  "all tests pass" command ran a subset.

## B. Inputs

### B1. Boundaries (FIX, SAME, FIND)

Empty, single element, exactly the limit, one past it, zero, negative, maximum
representable, very large. For strings: empty, whitespace only, non-ASCII,
combining characters, invalid UTF-8, embedded NUL, newline variants.

- **Failure signature:** an exception, wrong output, or a silently different
  result from the pre-change code on an input inside the claim's stated scope.

### B2. Shape variations (FIX, SAME)

Same semantic input in a different shape: optional field absent vs `null` vs
empty, trailing slash, mixed case, duplicate keys, reordered fields, different
numeric types for the same value.

- **Failure signature:** the fix applies to one shape only; the bug reproduces
  under another shape that the original report would also cover.

### B3. The reported reproduction, exactly (FIX)

Run the bug report's own reproduction steps, unmodified, not a reconstruction
of them. If the report has none, build one that reproduces on the pre-fix
code first, then run it on the fixed code.

- **Failure signature:** the reconstructed case passes but the original one
  does not; or the reproduction never failed on the pre-fix code, so it tests
  nothing.

### B4. Hostile input (FIX, FIND)

Inputs that reach a shell, SQL, filesystem path, regex, deserializer, or
template: quotes, separators, `..`, metacharacters, pathological regex input.

- **Failure signature:** the input is interpreted rather than treated as data,
  or processing time explodes.

## C. Callers and surface

### C1. Caller enumeration (LOCAL, SAME)

Search for every use of each changed symbol: literal name, re-exports,
aliases, string-keyed lookups, reflection, config files, templates, generated
code, and other languages in the repo that bind to it.

- **Failure signature:** a caller exists that the change did not consider,
  and running or tracing it shows a changed outcome.

### C2. Signature and contract drift (LOCAL, SAME)

For each changed function: argument defaults, accepted types, return type and
shape, raised exceptions, nullability, ordering guarantees, mutation of
arguments.

- **Failure signature:** a caller depends on a contract the change altered
  (relies on the old exception type, on a stable ordering, on the argument not
  being mutated).

### C3. Interface boundary (LOCAL)

Build, type-check, lint, generate: anything that consumes the changed code
beyond the test suite. Public API snapshots, schema files, docs generated from
source, serialized formats.

- **Failure signature:** a check outside the test suite fails, or a generated
  artifact changes when the claim said nothing else would.

## D. State, order, and environment

### D1. Repeat and reorder (FIX, SAME)

Run the operation twice in a row, in reverse order with its siblings, and
after a previous failure. Fresh process vs. warm process.

- **Failure signature:** the second run differs from the first (cache, global,
  leftover file, consumed iterator).

### D2. Concurrency (FIX, FIND)

Run the operation in parallel with itself or with its neighbor on shared
state. For a fix to a race, run the reproduction many times, not once.

- **Failure signature:** a different result, a deadlock, a corrupted shared
  structure, or a reproduction that still fails at a lower rate.

### D3. Environment (FIX, LOCAL)

Different working directory, unset environment variables, another locale or
time zone, CRLF checkouts, read-only files, a clean checkout without local
untracked files, the CI image rather than the dev machine.

- **Failure signature:** the behavior holds only under the environment the fix
  was developed in.

### D4. Versions (FIX, LOCAL)

The dependency versions the project actually pins or allows, and the runtime
versions it supports, not only the one installed locally.

- **Failure signature:** the fix relies on behavior of a version the project
  does not guarantee.

## E. Error paths

### E1. Dependency failure (FIX, SAME, FIND)

Make each external call the code makes fail: timeout, non-zero exit, missing
file, permission denied, malformed response.

- **Failure signature:** an unhandled exception, a swallowed error, a partial
  write, or a different error than the pre-change code produced where the
  claim is SAME.

### E2. Mid-operation interruption (FIX, FIND)

Fail halfway through a multi-step operation and inspect the state left behind.

- **Failure signature:** inconsistent state, a lock not released, a temp file
  not cleaned up, a retry that double-applies.

### E3. Resource limits (FIX, MEAS)

Large input, many iterations, a nearly-full disk or small memory limit, slow
I/O.

- **Failure signature:** a hang, an out-of-memory, or quadratic behavior where
  the claim asserts linear.

## F. Root-cause claims

### F1. Removal test (CAUSE)

Remove or neutralize only the claimed cause, with no other change, and run the
reproduction.

- **Failure signature:** the symptom persists. The claimed cause is at most a
  contributor.

### F2. Reintroduction test (CAUSE)

With the fix in place, reintroduce the claimed cause alone.

- **Failure signature:** the symptom does not return. The fix works for
  another reason, and the real cause is still untreated.

### F3. Alternative-source search (CAUSE)

Search for other paths that could produce the same symptom: other writers to
the same state, other callers producing the same error message, other places
the same wrong value could originate.

- **Failure signature:** another path reproduces the symptom independently of
  the claimed cause.

### F4. Timeline check (CAUSE)

When the symptom appeared, which commit introduced it (`git bisect`, `git log
-S`), and whether the claimed cause was present before the symptom was.

- **Failure signature:** the claimed cause predates the symptom and nothing
  else about it changed. Something else triggered it.

## G. Behavior preservation

### G1. Before/after diff on a corpus (SAME)

Run both versions on the same set of inputs — recorded fixtures, a sample of
real data, generated cases — and diff the full output including error output,
exit codes, and side effects on the filesystem.

- **Failure signature:** any difference the claim did not announce, including
  ordering, formatting, and error wording that callers may match on.

### G2. Error behavior diff (SAME)

Feed both versions the same invalid inputs.

- **Failure signature:** a different exception type, message, or exit code; an
  input that was rejected is now accepted, or the reverse.

### G3. Side-effect inventory (SAME, LOCAL)

List every write the old code performs — files, logs, metrics, network calls,
database rows, global state — and check each is still performed, in the same
order where order was observable.

- **Failure signature:** a write dropped, reordered, duplicated, or added.

## H. Measured properties

### H1. Same harness, many runs (MEAS)

Measure before and after with the same tool, same inputs, same machine state,
enough runs to see the variance.

- **Failure signature:** the claimed improvement is inside the run-to-run
  noise, or depends on a warm cache the "before" run did not have.

### H2. Flakiness by repetition (MEAS, FIX)

For "no longer flaky", run the test many times (tens to hundreds) in the
conditions under which it flaked, including under load.

- **Failure signature:** any failure. A single green run is no evidence
  against a probabilistic failure.

### H3. Scaling input (MEAS)

Measure at several input sizes, not one.

- **Failure signature:** the improvement holds at the measured size and
  reverses or vanishes at others.

## I. Review findings

### I1. Reach the line (FIND)

Construct an input or state that reaches the line the finding names, from a
real entry point, and show what happens there.

- **Failure signature for the finding (finding REFUTED):** no real path
  reaches the line, or the condition the finding assumes cannot hold there
  (a guard upstream, a type that excludes the value).
- **Failure signature for the code (finding CONFIRMED):** the path exists and
  the predicted wrong behavior occurs.

### I2. Pre-existing or introduced (FIND)

Run the same reproduction on the base branch.

- **Outcome:** the defect is pre-existing (true, but not this change's
  regression) or introduced by the change. Report which; it changes what the
  finding asks for.
