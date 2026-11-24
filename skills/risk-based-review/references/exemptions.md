# Exemptions

An exemption is a claim that a file's change cannot alter what the product
does. Each entry states the claim, the check that proves it, and the cases
that break it. Run the check; a claim that was only read is not proved.

Exemptions A–D are proved per file and give **NONE**. Exemption E is proved for
the dependency files as a group and gives **LOW** at best.

Run each command against the head commit recorded in Step 1. A check that
needs a tool or a network that is not available is NOT RUN, and the file is
not exempt.

## A. Docs-only

**Claim:** the file is prose for human readers, and no tool turns it into
behavior.

**Check:**

```bash
git grep -n -F -e '<path>' -e '<base name>' <head> -- . ':!<path>'
```

Read each hit that is about this file: a link from other prose is fine; a
reference from build configuration, source, a generator, or a package manifest
is not. Then confirm that neither the file nor the changed hunk is one of the
cases below.

**Breaks when the file is:**

- **A specification tools consume** — OpenAPI, protobuf, JSON Schema, GraphQL
  SDL. It generates code or validates traffic. Classify as Source, and see
  "Public contracts" in the catalog.
- **Instructions an agent or tool executes** — `SKILL.md`, `CLAUDE.md`,
  `AGENTS.md`, prompt templates, editor rule files. The prose is the program.
- **Input to release tooling** — changeset files, changelog fragments, or
  front matter that drives version bumps, routing, or publication.
- **Text the product shows** — email templates, message catalogs, in-app help.
  Source.
- **Steps a reader will run** — install commands, download URLs, checksums,
  version pins, release and recovery runbooks. The reader is the interpreter:
  a changed source or command changes what runs on their machine. Not NONE;
  MODERATE, and HIGH (Severe) when the hunk changes where code is fetched from
  or what is run with elevated rights.
- **A legal or policy statement** — `LICENSE`, terms, privacy policy, security
  policy. No runtime effect, but the wording is a commitment: Outside, HIGH.
- **Documentation with executed examples** — doctests. Classify as Tests.
- **The documentation site's code or configuration.** Source.

**Comment-only changes in a source file** are docs-only when the diff is empty
after comments are stripped and no changed comment is a directive: lint and
type suppressions (`eslint-disable`, `# type: ignore`, `@ts-expect-error`,
`# noqa`, `//nolint`), build constraints and generators (`//go:build`,
`//go:generate`), pragmas and magic comments (`# frozen_string_literal`,
`/* webpackChunkName */`, `#pragma`), coverage exclusions. A changed directive
is a Source change; a suppression added is also "Weakened verification".

## B. Tests-only

**Claim:** the file is loaded only by the test runner, does not ship, and the
change does not reduce what the suite verifies.

**Check:**

1. The path matches the project's test configuration — read the runner's
   include patterns, do not infer from the name.
2. No non-test code imports it:
   ```bash
   git grep -n -E -e '<module name or path>' <head> -- . ':!<test paths>'
   ```
3. The change does not weaken the suite. In the diff, look for:
   - removed test cases or assertions,
   - added skip markers (`.skip`, `xit`, `@pytest.mark.skip`, `t.Skip`,
     `@Ignore`, `#[ignore]`) or focus markers (`.only`, `fit`),
   - changed snapshot or golden files,
   - loosened expectations (an exact match replaced by a pattern, a widened
     tolerance, a raised timeout),
   - lowered coverage thresholds.

**Breaks when:**

- **The suite is weakened** (step 3). Production behavior is still untouched,
  but the file is not NONE: report it under "Weakened verification".
- **A snapshot changed and no source did.** Something else changed the output
  — a dependency, the environment, nondeterminism. Ask what.
- **The file is test infrastructure, not a test** — runner configuration, CI
  test jobs, coverage settings. Classify as Pipeline.
- **The helper is published** — a testing utility other packages import is a
  public contract.
- **The fixture is loaded into a real environment** as seed data. Classify as
  Data.

## C. Formatting-only

**Claim:** the program is the same before and after; only its layout changed.

**Check — use the strongest one available:**

1. **Reproduce it.** In a scratch worktree of the base, run the project's
   formatter at the project's pinned version and configuration over the
   changed files, and compare the result with the head:
   ```bash
   git worktree add <dir> <base>
   cd <dir> && <format command> <files> && git diff --exit-code <head> -- <files>
   ```
   An empty diff proves the change is exactly the formatter's output. A
   non-empty one is the **residue**: the real change with the reformatting
   taken out. The file is not exempt, but Step 5 reads the residue instead of
   the full diff.
2. **Compare a normalized form** of base and head: a syntax-tree dump,
   compiler output, bytecode, or the built artifact.
3. **Whitespace-only diff:**
   ```bash
   git diff -w --exit-code <base>...<head> -- <files>
   ```
   This is a proof only where whitespace carries no meaning. It is not one for
   Python, YAML, Makefiles, Markdown, or other indentation-sensitive formats,
   nor for text inside templates, JSX, multi-line strings, and heredocs in any
   language.

Reading the diff and judging that it looks like formatting is not a check. A
large reformat is the easiest place to carry one real change past a reviewer.

**Breaks when the diff contains:**

- **Reordered imports, includes, or statements.** Order can decide which side
  effect runs first or which definition wins. Read those hunks as Source.
- **Lint auto-fixes.** They rewrite code, not layout.
- **Line-ending or encoding changes** — CRLF in a shell script, a byte-order
  mark. Check 1 applies; check 3 hides them.
- **File mode changes** — visible in `git diff --summary`. An executable bit
  is behavior.
- **Renames or moves.** A path is an import target and sometimes a public
  contract; see "Claimed no-ops" in the catalog.
- **A formatter version or configuration change.** That hunk is Pipeline; the
  reformatting it causes can still be proved by check 1.

## D. Generated output

**Claim:** the file is exactly what the project's generator produces from
inputs at the head.

**Check:** run the generation command the project documents, at the head, and
compare:

```bash
<generate command> && git diff --exit-code -- <generated paths>
```

For vendored code, compare against the upstream release the manifest names.

**When it holds**, the generated file needs no reading: the risk is in the
generator's input and version, which are triaged as their own files.

**Breaks when:**

- **Regeneration differs.** The file was edited by hand or built from
  something else. HIGH, Outside — nobody can say what it contains from the
  diff.
- **There is no way to regenerate** — a binary, a minified bundle, a vendored
  blob with no named upstream. Same.
- **The generator is not pinned**, so the output depends on the machine that
  ran it.

## E. Version-only dependency update

**Claim:** only dependency versions changed, each within a range its upstream
promises is compatible, and the new code comes from where the old code did.

Who opened the change is not part of the claim. Anyone with push access can
add a commit to a bot's branch; what was changed is read from the diff, and
what the new versions are is read from the registry.

**Conditions — check each, for every package that changed:**

1. **Only version pins changed.**
   ```bash
   git diff --name-only <base>...<head>
   ```
   Every changed file is a manifest, a lockfile, or a tool-version file, and
   every changed line in it is a version, a hash, or a resolved source. Any
   other file or hunk is triaged as its own kind.
2. **The bump is one upstream promises is compatible.** On a major version of
   1 or higher: patch or minor. On `0.x`: patch only — a `0.x` minor is where
   breaking changes go. The promise must be stated by the package or be the
   convention of its registry; calendar-versioned packages, compilers, and
   language toolchains make none. Where it cannot be established, the
   condition fails. A bump that skips several releases is judged by the
   largest step.
3. **The lockfile delta is explained by the manifest delta.** List what the
   lockfile added, removed, and changed, and look for:
   - packages that are new to the tree,
   - transitive dependencies that jumped a major version,
   - new install or lifecycle scripts,
   - a changed registry host, or a git, URL, or tarball source,
   - a changed integrity hash for an unchanged version.

   With no lockfile, the installed tree is not determined by the repository
   and this condition cannot be checked.
4. **The release is past the project's cooldown** — the bot's release-age
   setting that applies to this package's datasource (Renovate
   `minimumReleaseAge`, Dependabot `cooldown`), or three days where none
   applies. Measure from the registry's publish time to the time of triage.
   Malicious releases are usually pulled within days, and they arrive as patch
   and minor bumps because those are the ones merged without reading. A
   version already superseded by a quick follow-up release fails too: ask what
   the follow-up fixed.
5. **The package's role in the product is not sensitive.** Sensitive roles:
   authentication, cryptography, TLS, sessions, sanitization, parsing or
   deserializing untrusted input; and anything that shapes the shipped
   artifact — compiler, bundler, code generator, base image. A tool that
   merely runs with the developer's credentials — a CLI, an installer, an
   agent — is not sensitive in this sense; conditions 3 and 4 are what guard
   it.

**Level:**

| Outcome | Level | Ask the reviewer to |
| --- | --- | --- |
| All five hold | LOW | — |
| 5 fails, the rest hold | MODERATE | Read the upstream release notes for changed defaults and behavior. |
| 2 fails | MODERATE; HIGH (Severe) in a sensitive role | Read the upstream changelog for breaking changes and check each call site that uses the changed API. |
| 3 fails on a new package only | MODERATE | Identify what the package is, who publishes it, and why the tree now needs it. |
| 3 fails on scripts, source, host, or integrity | HIGH (Severe, Outside) | Establish where the code comes from and what the script runs before anything installs it. |
| 3 cannot be checked (no lockfile) | MODERATE | Check each direct dependency's registry metadata for scripts and source; note that the transitive tree is unverified. |
| 4 fails or cannot be checked | MODERATE | Check the release date and the registry's advisories for that version. |

When several rows apply to one package, it takes the highest. The table gives
the level for the dependency files; they do not go on to Step 5 unless a row
or an adjustment below names a catalog entry.

**Adjustments:**

- **Tooling that neither ships nor shapes the artifact** — linters,
  formatters, test runners, type stubs. Where the project's checks run the
  tool, a break shows up as a failing check, so condition 2 does not apply.
  Conditions 3 and 4 still do: the code runs on developer machines and in CI
  with credentials.
- **Grouped updates.** Assess each package on its own row; the group takes the
  highest level.
- **CI actions and base images.** A tag is mutable, so a tag reference carries
  no version promise: condition 2 holds only for a reference pinned to a
  commit SHA or image digest. A job that holds secrets or write permissions is
  also "Build, CI, and release" in the catalog.
- **No automated checks exercise the dependency,** or their result on the
  head cannot be observed. LOW rests on those checks; without them the update
  is MODERATE.
