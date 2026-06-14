---
name: serena-semantic-search
description: Use Serena's semantic, symbol-level read tools (find_symbol, get_symbols_overview, find_referencing_symbols, search_for_pattern) as a smarter, more token-efficient alternative to plain grep/ripgrep and reading whole files. Use when searching a codebase, locating where a symbol is defined or used, mapping a file's structure, tracing call sites, or understanding code relationships — and when the Serena MCP server is available.
license: CC-BY-4.0
---

# Serena as a Semantic Grep

## Why

When the `serena` MCP server is available, its language-server-backed read tools beat plain `grep`/`rg` + reading whole files for **understanding code**:

- **More accurate** — `find_symbol` / `find_referencing_symbols` resolve symbols through the language server, so they match definitions and references by meaning, not by text. They skip comments, strings, and same-named-but-unrelated tokens that trip up regex.
- **More token-efficient** — `get_symbols_overview` returns a file's symbol skeleton instead of its full text; `find_symbol` can return just a signature or a single method body. You read the relevant slice, not the whole file.

In this setup Serena is configured **read-only** (all write/edit tools are disabled at the config level). Treat it as a query engine: reach for it to *read and locate* code, never to *change* it.

## Reach for Serena vs. plain grep/Read

| Intent | Use |
| --- | --- |
| "Where is `Foo` / `Foo.bar` defined?" | `find_symbol` |
| "Who calls / references `Foo.bar`?" | `find_referencing_symbols` |
| "What's in this file / what are its classes & methods?" | `get_symbols_overview` |
| "Show me the body of just this one method" | `find_symbol` with `include_body` on that symbol |
| Regex / substring across code (no symbol resolution needed) | `search_for_pattern` (Serena) or `Grep` |
| Plain-text search: logs, docs, config, comments, commit messages | `Grep` / `Bash` (Serena gives no edge here) |
| You already know the file and need its literal contents | `Read` (or Serena `read_file`) |

Rule of thumb: **if the question is about a symbol or a code relationship, start with Serena; if it's about raw text, use Grep.**

## Prerequisites

Serena's symbolic tools need an **activated project with a ready language server**:

1. Make sure the project is activated (`activate_project` / `check_onboarding_performed`). The `mcp.json` here launches Serena with `--project .`, so the working directory is usually already active.
2. Symbolic tools only work for **LSP-supported languages** (Python, TS/JS, Go, Rust, Java, C/C++, and ~40 more). For an unsupported language or non-code file, fall back to `search_for_pattern` / `Grep`.
3. The first symbolic call on a project can be slow while the language server indexes. A "symbol not found" right after activation often means indexing isn't finished, not that the symbol is absent — retry, or cross-check with `search_for_pattern`.

## Do NOT edit through Serena

All Serena write tools (`replace_symbol_body`, `insert_after_symbol`, `create_text_file`, `execute_shell_command`, memory writes, …) are intentionally disabled in this environment. Make every code change with the host agent's own **Edit / Write / Bash**. If a Serena edit tool ever appears callable, do not use it — it is redundant with native editing and adds round-trips.

## Reference

For the full read-tool catalog, language-support details, fallback patterns, and troubleshooting (stale index, monorepos, generated code), see [references/tool-reference.md](references/tool-reference.md).
