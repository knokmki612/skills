# Risk catalog

Risk factors grouped by category. Each entry gives:

- **Find it** — path signals (where the file is) and content signals (what the
  hunk does). Match both; content signals are the ones path matching misses.
- **Why** — what goes wrong, and which reason applies (Outside, Irreversible,
  Emergent, Severe — defined in `SKILL.md`).
- **Ask** — question templates. Rewrite each in terms of the actual hunk
  before putting it in the report.
- **Verify** — how the answer is found.
- **Level** — the default, and what raises or lowers it.

The catalog lists where the four reasons usually occur. A hunk that matches no
entry is MODERATE, or HIGH if one of the reasons applies anyway.

An entry's reasons apply to what the hunk changes, not to every file under
its paths. Read the **Level** line before assigning HIGH: most entries are
MODERATE for a narrowing, a purely additive change, or a change confined to
how checks run.

## 1. Declared permissions and exposure

Files that declare what the product may do and how it is reachable.

- **Find it — paths:** application manifests (`AndroidManifest.xml`,
  `Info.plist`, entitlements, a browser extension's `manifest.json`, desktop
  app capability files, sandbox permissions for packaged apps); access policy
  (IAM, RBAC, security contexts, OAuth scope lists); network exposure (CORS,
  CSP, security groups, ingress rules); workflow `permissions:` blocks.
- **Find it — content:** an added permission, scope, host pattern, or
  capability; a component made exported or externally reachable; a relaxed
  transport rule (cleartext allowed, a TLS exception); debug or backup flags
  turned on; a wildcard where a list was.
- **Why:** the change widens what a compromised or buggy product can reach
  (Severe). A granted permission is often not withdrawn by a revert: users
  consented, stores reviewed, tokens were issued (Irreversible). Whether a
  store or administrator will accept it is not in the repository (Outside).
- **Ask:** Which feature needs the new permission, and is it the narrowest
  one that works? What can now reach the newly exposed component, and does it
  authenticate the caller? Will this trigger a store review, a re-consent
  prompt, or disable auto-update for existing users? If a permission was
  removed, what still uses it?
- **Verify:** a before/after list of declared permissions; the platform's
  documentation on what the change triggers; a human install or upgrade test
  on a release candidate.
- **Level:** HIGH for an addition or widening, measured against what was in
  effect before. A removal or narrowing is MODERATE — the question is what
  breaks, and that is in the repository. An explicit declaration that
  restricts a broader default is a narrowing, though the diff shows it as
  added lines.

## 2. Security logic

Code that decides who may do what and what input is trusted.

- **Find it — paths:** authentication, sessions, authorization and policy
  code, middleware and interceptors, input validation and sanitization, output
  encoding, signature and certificate verification, cryptographic and TLS
  configuration, secret handling, rate limiting, audit logging.
- **Find it — content, in any file:** a removed or reordered guard clause; a
  changed comparison or boolean operator in a condition; a changed default
  for a security-relevant option; middleware or route registration reordered;
  a broadened `catch` or a swallowed error near a check; a changed regular
  expression, allowlist, or schema; verification made optional or skippable;
  a hard-coded credential, key, or token; data from a request flowing into a
  query, command, file path, URL, redirect, template, or deserializer.
- **Why:** these fail silently and in the permissive direction — the product
  keeps working while accepting what it should reject (Severe).
- **Ask:** When the check cannot decide — an error, a timeout, a missing
  value — does the code deny or allow? Is there a test for the rejected case,
  and does it fail without the check? Which requests reach the changed code
  without passing through the check? Did the set of accepted inputs grow, and
  was that intended? Is a secret now logged, returned, or committed?
- **Verify:** negative tests for the denied path; the revert check on those
  tests; a security review by someone qualified in the area.
- **Level:** HIGH.

## 3. Reach across boundaries

A change whose effects land in code the diff does not show.

- **Find it — paths:** shared libraries and utilities, base classes, common
  types and schemas, global state, shared styles and design tokens, anything
  another package or application imports.
- **Find it — content:** a changed signature, return shape, thrown error,
  default argument, or side effect of a symbol that other code can see; a
  changed global, singleton, or environment read.
- **Why:** the callers are not in the diff, so the diff cannot show whether
  they still work. The more callers, and the further from the author's own
  package, the less the author is likely to have checked.
- **Ask:** How many callers does the changed symbol have, and in which
  packages? Which of them rely on the behavior that changed? Are there users
  that search would miss — string lookups, reflection, dependency injection,
  configuration-driven dispatch, consumers in other repositories? Which
  callers have tests that exercise the changed behavior?
- **Verify:** a caller search, recorded with its command and count; the test
  suites of the consuming packages, not only the changed one.
- **Level:** MODERATE when every caller is inside the repository and can be
  found by search. HIGH when callers owned by someone else exist outside the
  repository, or callers cannot be enumerated (Outside). The project's own
  deployment targets are not callers in this sense.

## 4. Public contracts

Anything a consumer depends on that cannot be changed in the same commit as
the consumer.

- **Find it — paths:** HTTP, GraphQL, and RPC schemas and handlers; a
  library's exported surface and published types; CLI flags, exit codes, and
  output formats; configuration file formats and environment variable names;
  event, message, and webhook payloads; database schemas read by more than
  one deployed version; persisted formats (files, caches, client-side
  storage); URLs.
- **Find it — content:** a field, flag, endpoint, or export removed or
  renamed; a type narrowed, a required field added, an optional one made
  required; a changed default, ordering, error code, or status code; behavior
  changed under an unchanged signature.
- **Why:** consumers the repository does not contain depend on it (Outside),
  and a published release or a stored record in the old format cannot be
  recalled (Irreversible).
- **Ask:** Is the change additive, breaking, or behavioral under the same
  signature? Who consumes this, and how will they learn of it? Does the
  version bump match the change? During a rolling deploy, does the old
  version still read what the new one writes, and the reverse? Is there a
  deprecation path, and are the changelog and documentation updated?
- **Verify:** a schema or API diff tool; contract and compatibility tests;
  confirmation from the consumers' owners.
- **Level:** HIGH for breaking and behavioral changes. MODERATE for a purely
  additive change that no existing consumer has to act on.

## 5. Data and irreversible operations

- **Find it — paths:** schema migrations, backfill and maintenance scripts,
  seed data loaded into real environments, infrastructure definitions for
  stateful resources, retention and deletion jobs.
- **Find it — content:** dropped or renamed columns, tables, or indexes; a
  type change or a new constraint on existing data; an `UPDATE` or `DELETE`
  without a bounded predicate; a migration with no down path; a destructive
  command against storage, queues, or buckets.
- **Why:** a revert of the commit does not bring the data back
  (Irreversible), and whether existing rows satisfy the new shape depends on
  production data (Outside).
- **Ask:** Can this be undone after it runs, and how? Does existing data
  satisfy the new constraint? Does the currently deployed code still work
  against the migrated schema, and the new code against the old one? How long
  does it hold a lock on a table of production size? Is there a backup, and
  has the restore been tried?
- **Verify:** a run against a copy of production-shaped data; a tested
  rollback; an accountable person's sign-off on the recovery plan.
- **Level:** HIGH.

## 6. Build, CI, and release

Definitions of what gets built, with which credentials, and where it goes.

- **Find it — paths:** CI workflows, build scripts, container files, release
  and deployment configuration, package publishing settings, signing
  configuration.
- **Find it — content:** changed job permissions, secrets, or triggers —
  especially triggers that run code from forks with repository credentials;
  an action, image, or tool referenced by a mutable tag; a download piped to
  a shell; changed build flags, targets, or the list of files that ship; a
  removed or weakened check job.
- **Why:** the pipeline holds credentials and decides what users receive
  (Severe), and a published artifact cannot be unpublished (Irreversible).
  Most of it runs only on merge or release, so the change's own checks do not
  exercise it (Emergent).
- **Ask:** What can this job now read or write, and who can trigger it? Does
  the shipped artifact contain the same files as before? Is every external
  action, image, and tool pinned to an immutable reference? Which check no
  longer runs, and what covered that ground?
- **Verify:** a dry run of the release; a diff of the built artifact's
  contents before and after; the platform's guidance for the trigger in use.
- **Level:** HIGH when a job gains write permissions or secrets, a trigger
  lets code from outside run with credentials, a mutable reference is
  introduced into a job that holds either, or the shipped artifact changes.
  MODERATE for the rest — a read-only check job, added or changed, is how
  checks are run. A removed check is also "Weakened verification".

## 7. Configuration and defaults

- **Find it — paths:** environment files and their templates, feature flag
  definitions, service configuration, limits and quotas.
- **Find it — content:** a changed timeout, retry count, pool size, limit, or
  threshold; a flag's default flipped; a new required setting with no
  default; a changed endpoint or resource name.
- **Why:** the effect depends on values and load that exist only in the
  running environment (Emergent), and often on settings kept outside the
  repository (Outside). A one-line change here can take effect everywhere at
  once.
- **Ask:** Which environments read this value, and is it overridden anywhere
  outside the repository? What happens to a running deployment that lacks the
  new setting? Does the new value hold at production load? Does flipping this
  flag turn on code that has not been through integration testing?
- **Verify:** a staged rollout; a check of each environment's actual values.
- **Level:** HIGH when it changes production behavior. MODERATE when it
  affects only development or test environments.

## 8. New features

- **Find it — paths:** new modules, routes, screens, commands, jobs.
- **Find it — content:** a new entry point wired into routing, navigation,
  scheduling, or a public surface; new code called from existing paths; new
  state, storage, or schema introduced alongside it.
- **Why:** new code has no history of working. Its tests encode the author's
  understanding of the requirement, so a misunderstanding passes them
  (Outside), and how it behaves with the rest of the product appears only
  when the whole is run (Emergent).
- **Ask:** Can a user reach the new code by default, or is it behind a flag
  that is off? Which existing paths now call into it, and what happens to
  them if it throws? Does it match the requirement as the requester
  understands it? What state does it create, and what happens to that state
  if the feature is withdrawn?
- **Verify:** human integration testing on a release candidate, against the
  requirement; a staged rollout.
- **Level:** HIGH when the feature is reachable by default or existing paths
  call into it. MODERATE when it is unreachable — but then the later change
  that turns it on is the HIGH one, under "Configuration and defaults". An
  addition small enough to be read whole and pinned by one test — a flag, an
  option, a helper — is not a feature in this sense: treat it as an additive
  change under "Public contracts".

## 9. Behavior that tests rarely capture

- **Find it — content:** changed locking, transaction boundaries, or
  isolation; changed cache keys, invalidation, or expiry; changed retry,
  timeout, or idempotency handling; changed ordering of asynchronous work;
  changed error handling on a failure path; a query or loop added to a hot
  path; time, timezone, or locale arithmetic.
- **Why:** the failure needs concurrency, load, or a fault to appear, so a
  passing suite says little (Emergent).
- **Ask:** What happens when two of these run at once, or one runs twice?
  What happens when the step in the middle fails? What is served while the
  cache is stale? How does the cost grow with data size?
- **Verify:** a test that forces the interleaving or the fault; a load test;
  a staged rollout with the relevant metric watched.
- **Level:** HIGH on shared or user-facing paths. MODERATE when confined to a
  path with a test that already forces the condition.

## 10. Money, personal data, and legal obligations

- **Find it — paths and content:** pricing, billing, tax, and payment code;
  rounding and currency conversion; collection, storage, logging, export, or
  deletion of personal data; consent handling; retention rules; license and
  terms text; third-party data sharing.
- **Why:** small errors are costly and may be reportable (Severe); what the
  law or a contract requires is not in the repository (Outside).
- **Ask:** Does any amount change for an existing customer? Does personal
  data now reach a log, an analytics event, or a third party it did not reach
  before? Is deletion still complete? Who decided the new wording or rule?
- **Verify:** reconciliation against known cases; review by the responsible
  domain owner.
- **Level:** HIGH.

## 11. Weakened verification

Changes that reduce what would catch the next mistake.

- **Find it — content:** tests removed, skipped, or loosened; snapshots
  regenerated; lint rules disabled or suppression comments added; types
  loosened to an escape hatch; coverage thresholds lowered; a check job
  removed or made non-blocking; alerts and monitors removed.
- **Why:** nothing fails today. The cost arrives with a later change that the
  removed check would have stopped — and the removal can be hiding a
  regression already present.
- **Ask:** What did the removed check verify, and what verifies it now? Was
  it failing — and if so, was the cause fixed or silenced? Does the
  regenerated snapshot differ in a way that was intended?
- **Verify:** the reason in the change itself; the removed test run against
  the current code.
- **Level:** MODERATE. HIGH when the weakened check covers an area from
  entries 1, 2, 4, 5, or 10.

## 12. Claimed no-ops

Changes presented as altering nothing: a behavior-preserving refactoring, a
rename or move, a dead-code removal, a revert.

- **Find it:** a description or commit message that says so, or a diff shaped
  like one — code moved, renamed, or deleted with nothing added. The statement
  is not evidence that nothing changes; it names the claim to test.
- **Why:** "nothing changes" is a claim, and the review tends to accept it
  instead of testing it. Dead code may be reached dynamically; a renamed path
  may be a public import; a revert may undo a change that later work or a
  migration now depends on.
- **Ask:** What shows the behavior is the same — which tests ran, and do they
  exercise the moved code? Is the removed code referenced by name anywhere,
  including configuration and strings? Is the old path or name part of a
  public contract? What was built on top of the change being reverted?
- **Verify:** a reference search for every removed or renamed name; the same
  tests passing before and after; a before/after comparison of output. Treat
  it as a claim to falsify, not to re-read.
- **Level:** MODERATE. Raise it by whichever of entries 3 and 4 the touched
  symbols fall under.

## 13. Instructions for agents and tools

Prose that a model or a tool executes: skill files, system prompts, agent
rule files, prompt templates.

- **Find it — paths:** `SKILL.md` and its references, `CLAUDE.md`,
  `AGENTS.md`, prompt and rule directories, front matter that decides when an
  instruction loads.
- **Find it — content:** a trigger or description widened or narrowed; scope
  extended to new inputs; a prohibition, a confirmation gate, or a
  verification step relaxed or removed; examples for a language or case
  dropped; two rules that now contradict each other.
- **Why:** what a model does with the text is known only by running it
  (Emergent), and a reader of the diff tends to judge the prose instead of
  the behavior. A relaxed gate in front of a destructive action is Severe.
- **Ask:** Does the instruction now load for requests where it was not asked
  for, or stop loading for the ones it was written for? What does the agent
  do at the relaxed rule in the case the old wording forbade? Do the
  unchanged sections still agree with the changed one? Who receives this
  without pinning a version?
- **Verify:** the instruction run on fixed inputs at the base and at the
  head, outputs compared; a trigger evaluation for a changed description.
- **Level:** HIGH when a trigger or scope widens, or a prohibition or gate is
  relaxed. MODERATE for rewording, regrouping, and added examples.

## Shape of the diff

Not a risk factor, but it limits how far any review can be relied on. Note it
under "To lower the level" when it applies:

- **Mixed concerns** — exempt files alongside risky hunks, or a refactoring
  with a behavior change inside it. Splitting lets the exempt part be proved
  and the risky part be read alone.
- **Size** — a diff too large to hold in one reading gets a shallower review
  at every line. Say so; do not let volume stand in for scrutiny.
- **Unreadable content** — binaries, submodule moves, oversized files. Either
  they are proved as generated output or they are HIGH.
