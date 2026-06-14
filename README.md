# skills

A collection of [Agent Skills](https://agentskills.io/home).

Each skill lives in its own directory under `skills/`, anchored by a `SKILL.md` file. Agentic tool (Such as Claude Code, Codex, etc.) loads a skill automatically when the task at hand matches its `description`.

## Available skills

| Skill | Description |
| --- | --- |
| [safety-deletion](skills/safety-deletion/) | Route every deletion inside a git-managed working tree through `git rm` / `git clean` instead of destructive commands (`rm`, `rmdir`, `shred`, …), keeping deletions recoverable through git history and the reflog. |
| [serena-semantic-search](skills/serena-semantic-search/) | Use Serena's semantic, symbol-level read tools as a smarter, more token-efficient alternative to plain grep and reading whole files when searching, navigating, and understanding a codebase. |

## Installation

Using [`gh skill`](https://github.blog/changelog/2026-04-16-manage-agent-skills-with-github-cli/) (requires GitHub CLI v2.90.0 or later):

```bash
# Preview a skill before installing
gh skill preview knokmki612/skills safety-deletion

# Install a skill
gh skill install knokmki612/skills safety-deletion
```

`gh skill` writes the skill to the correct directory for your agent. Pass `--agent` (e.g. `claude-code`) and `--scope` (`user` or `project`) to control the target, and pin a version with `--pin <tag-or-sha>`.

Once installed, the agent reads the `description` from each `SKILL.md` and invokes the skill automatically when a relevant operation comes up.

## Skill structure

```
skills/
└── some-skill/
    ├── SKILL.md              # frontmatter (name / description) + body
    └── references/
        └── edge-cases.md     # supplementary docs, loaded only when needed
```

- The `name` and `description` in the `SKILL.md` frontmatter tell Agentic AI when to use the skill.
- Files under `references/` are linked from the body and pulled in only when the relevant case applies.

## License

- Code (skill logic and scripts): [MIT License](LICENSE)
- Documentation (`SKILL.md`, `references/`, and other prose): [CC BY 4.0](LICENSE-docs)
