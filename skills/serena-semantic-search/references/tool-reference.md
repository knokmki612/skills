# Serena Read-Tool Reference

Consult this when you need more than the SKILL.md decision table — the full
read-only tool catalog, language coverage, and how to recover when a symbolic
lookup comes back empty.

## Read-only tool catalog

These are the Serena tools that stay enabled under the read-only config. Names
follow `oraios/serena`; if a call 404s the upstream tool may have been renamed —
cross-check against the `excluded_tools` list in the dotfiles `serena_config.yml`.

### Symbol navigation (language-server backed)

| Tool | What it answers | Notes |
| --- | --- | --- |
| `find_symbol` | Locate a symbol by name path (`Class/method`, `module.func`). | Set `include_body` to get the source of just that symbol. Use `depth` to also pull children (e.g. a class's methods). Supports `substring_matching` for fuzzy names. |
| `get_symbols_overview` | The top-level symbol skeleton of a file or directory. | Best first move on an unfamiliar file — cheaper than reading it whole. |
| `find_referencing_symbols` | Every place a given symbol is used. | The semantic answer to "who calls this?" — far more reliable than grepping the name. |

### Pattern & file access (no LSP required)

| Tool | What it answers | Notes |
| --- | --- | --- |
| `search_for_pattern` | Regex across the codebase. | The fallback when a language isn't LSP-supported, or when you're searching for text rather than a resolved symbol. Supports include/exclude globs and context lines. |
| `list_dir` | Directory contents. | Honors project gitignore; good for orienting in a repo. |
| `find_file` | Files matching a name/glob. | |
| `read_file` | Raw file contents. | Equivalent to the host `Read`; prefer `get_symbols_overview` first to avoid pulling a whole file. |

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
