# Serena Read-Tool Reference

Consult this when you need more than the SKILL.md decision table — the full
read-only tool catalog, language coverage, and how to recover when a symbolic
lookup comes back empty.

## Read-only tool catalog

These are the Serena read tools that stay enabled under the read-only config.
Names are verified against `oraios/serena` `src/serena/tools/symbol_tools.py` and
`file_tools.py`; if a call 404s the upstream tool may have been renamed —
cross-check against the `excluded_tools` list in the dotfiles `serena_config.yml`.

### Class A — semantic (language-server backed, no grep/sed/awk equivalent)

These are the reason to use Serena. A regex or line tool cannot reproduce them.

| Tool | What it answers | Notes |
| --- | --- | --- |
| `find_symbol` | Locate a symbol by name path (`Class/method`, `module.func`). | `include_body` returns just that symbol's source; `depth` pulls children (a class's methods); `substring_matching` for fuzzy names; `relative_path` scopes the search. |
| `find_declaration` | The declaration/definition behind a usage. | Resolves through imports/scope — go-to-definition, not text match. |
| `find_implementations` | Symbols that implement/override a given one. | Type-hierarchy query (`include_info` for extra detail). |
| `find_referencing_symbols` | Every place a symbol is used. | The semantic "who calls this?" — far more reliable than grepping the name; skips comments, strings, and namesakes. |
| `get_symbols_overview` | The top-level symbol skeleton of a file/dir. | Best first move on an unfamiliar file — cheaper than reading it whole (`depth`, `max_answer_chars`). |
| `get_diagnostics_for_file` | Compiler/linter diagnostics for a file, grouped by severity. | Pure LSP — impossible from text tools (`min_severity`, line range). |
| `get_diagnostics_for_symbol` | Diagnostics for one symbol (optionally its referencers). | `check_symbol_references`, `min_severity`. |

### Class B — text-equivalent (duplicates native search/read/list/find — avoid routing through Serena)

Enabled, but they overlap tools you already have. Going through the MCP server
adds a round-trip with no semantic gain, so prefer the native column. The native
column is representative, not exhaustive.

| Serena tool | Overlaps | Prefer instead (e.g.) |
| --- | --- | --- |
| `search_for_pattern` | regex/substring search | `Grep` (ripgrep) / `git grep` |
| `read_file` | reading a file or line range | `Read` / `sed -n 'A,Bp'` / `cat` |
| `list_dir` | directory listing | `ls` / `Glob` |
| `find_file` | file-name/glob search | `find` / `fd` / `Glob` |

The one time Class B earns its keep: you need to search/read *inside the activated
project's ignore-aware view* and you're already in a Serena turn — otherwise the
native tools are lighter.

### Reads with no Serena counterpart (always native)

These never conflicted with Serena because it has no tool for them — keep them on
the standard utilities:

- **Field / column extraction, read-time aggregation** → `awk`, `cut` (Serena
  returns whole lines/symbols, not parsed columns or computed values).
- **Structured-format reads** → `jq` (JSON), `yq` (YAML). Serena's symbol tools
  are LSP/code-oriented and do not parse config documents semantically.

Note `ed` is a line *editor*, not a read tool; for reading prefer `Read` / `sed`
/ `cat`. Within this read-only skill, editing is out of scope entirely.

### Project & memory (read side)

| Tool | What it answers |
| --- | --- |
| `activate_project` | Make a project the active one for symbolic tools. |
| `check_onboarding_performed` | Whether Serena has onboarded this project. |
| `list_memories` / `read_memory` | Read Serena's stored project notes. Memory *writes* are disabled. |

## Language support

Symbolic tools depend on a language server, available for ~40+ languages
including Python, TypeScript/JavaScript, Go, Rust, Java, C/C++, C#, Ruby, PHP,
Kotlin, Swift, and more. For anything without LSP support — plain text, Markdown,
YAML/JSON config, shell, Dockerfiles, logs — the symbolic tools add nothing;
use `search_for_pattern` or the host `Grep`.

## find_symbol vs. search_for_pattern

Serena's own framing: `find_symbol` is *semantic* retrieval at the symbol level;
`search_for_pattern` is *low-level* (line numbers, primitive regex). Decide by
what you actually have in hand:

- You have a **symbol name** and want its definition, body, or references →
  `find_symbol` / `find_referencing_symbols`. The result is structural and
  precise, and won't drown you in string/comment matches.
- You have a **regex or a literal substring** (an error string, a config key, a
  TODO marker, a non-code file) → `search_for_pattern` or `Grep`.
- You're **exploring** a file or module you don't know yet →
  `get_symbols_overview` first, then drill in with `find_symbol`.

## Recovering from empty results

A symbolic lookup returning nothing usually means one of:

1. **Index not ready.** First symbolic call after activation can race the
   language server. Wait briefly and retry, or confirm the project is active
   with `check_onboarding_performed`.
2. **Wrong name path.** `find_symbol` matches name paths like `Outer/inner`, not
   free text. Try `substring_matching`, or `get_symbols_overview` on the
   suspected file to read off the exact name path.
3. **Unsupported language / generated code.** No language server, or the symbol
   lives in generated/vendored output the server doesn't index. Fall back to
   `search_for_pattern`.
4. **Monorepo / multi-root.** The symbol may live in a sub-project that isn't the
   active one. Activate the right project, or widen `search_for_pattern` globs.

When in doubt, corroborate a symbolic result with a quick `search_for_pattern`
on the same name — agreement gives confidence, disagreement flags an indexing or
naming issue.

## Boundaries

- **Never edit via Serena.** Write tools are disabled by design; use the host
  agent's Edit/Write/Bash. This file documents *reading* only.
- **Don't double-search.** If Serena answered the question, don't re-run the same
  query as a `Grep` "just to check" unless an empty/suspicious result warrants
  corroboration (above). Pick the right tool once.
