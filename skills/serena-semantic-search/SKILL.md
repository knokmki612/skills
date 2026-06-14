---
name: serena-semantic-search
description: Use Serena's semantic, symbol-level read tools instead of plain grep or reading whole files when working in a codebase: locating where a symbol is defined or used, finding implementations, mapping a file's structure, tracing call sites, or surfacing compiler/LSP diagnostics. Needs the Serena MCP server. For plain text search, line-range reads, or listing files, prefer the existing grep/sed/Read tools.
license: CC-BY-4.0
---

# Serena as a Semantic Grep

## Why

When the `serena` MCP server is available, its language-server-backed read tools beat plain `grep`/`rg` + reading whole files for **understanding code**:

- **More accurate** — `find_symbol` / `find_referencing_symbols` resolve symbols through the language server, so they match definitions and references by meaning, not by text. They skip comments, strings, and same-named-but-unrelated tokens that trip up regex.
- **More token-efficient** — `get_symbols_overview` returns a file's symbol skeleton instead of its full text; `find_symbol` can return just a signature or a single method body. You read the relevant slice, not the whole file.

In this setup Serena is configured **read-only** (all write/edit tools are disabled at the config level). Treat it as a query engine: reach for it to *read and locate* code, never to *change* it.

## Serena's read tools split into two classes

Serena's read-only tools partition cleanly by whether `grep`/`sed`/`ed`/`Read`
can already do the job. Only the first class is the reason to use Serena.

**Class A — semantic, no text-tool equivalent → use Serena.** A language server
resolves these; regex/line tools structurally cannot reproduce them.

| Intent | Tool | Why grep/sed/ed can't |
| --- | --- | --- |
| "Where is `Foo` / `Foo.bar` defined?" | `find_symbol` (add `include_body` for the source) | grep matches the *text* `Foo`, not the resolved symbol (misses re-exports, picks up comments/strings/namesakes). |
| "Jump to the definition behind this call." | `find_declaration` | Needs import/scope resolution. |
| "What implements this interface / overrides this method?" | `find_implementations` | Type-hierarchy knowledge. |
| "Who calls / references `Foo.bar`?" | `find_referencing_symbols` | grep finds the name string, not true references. |
| "Outline this file's classes & methods." | `get_symbols_overview` | No structural outline from text. |
| "What does the compiler/linter flag here?" | `get_diagnostics_for_file` / `get_diagnostics_for_symbol` | Pure LSP — impossible from text. |

**Class B — text-equivalent → stay on grep/sed/ed/native, don't route through Serena.**
These duplicate tools you already have; going through the MCP server only adds a
round-trip with zero semantic gain.

| Task | Serena tool (avoid) | Use instead |
| --- | --- | --- |
| Regex / substring search across files | `search_for_pattern` | `Grep` (ripgrep) / `git grep` |
| Read a file or a line range | `read_file` | `Read` / `sed -n 'A,Bp'` / `cat` |
| List a directory | `list_dir` | `ls` / `Glob` |
| Find files by name/glob | `find_file` | `find` / `fd` / `Glob` |

Rule of thumb: **reach for Serena only for symbol resolution, reference/impl
graphs, and diagnostics (Class A). For text search, line-range reads, listing,
and file-finding, the existing grep/sed/ed/native path wins — those are exactly
the operations that do *not* conflict, so keep them where they are.**

## Prerequisites

Serena's symbolic tools need an **activated project with a ready language server**:

1. Make sure the project is activated (`activate_project` / `check_onboarding_performed`). The `mcp.json` here launches Serena with `--project .`, so the working directory is usually already active.
2. Symbolic tools only work for **LSP-supported languages** (Python, TS/JS, Go, Rust, Java, C/C++, and ~40 more). For an unsupported language or non-code file, fall back to `search_for_pattern` / `Grep`.
3. The first symbolic call on a project can be slow while the language server indexes. A "symbol not found" right after activation often means indexing isn't finished, not that the symbol is absent — retry, or cross-check with `search_for_pattern`.

## Do NOT edit through Serena

All Serena write tools (`replace_symbol_body`, `insert_after_symbol`, `create_text_file`, `execute_shell_command`, memory writes, …) are intentionally disabled in this environment. Make every code change with the host agent's own **Edit / Write / Bash**. If a Serena edit tool ever appears callable, do not use it — it is redundant with native editing and adds round-trips.

## Reference

For the full read-tool catalog, language-support details, fallback patterns, and troubleshooting (stale index, monorepos, generated code), see [references/tool-reference.md](references/tool-reference.md).
