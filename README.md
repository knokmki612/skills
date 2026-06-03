# skills

A collection of [Agent Skills](https://agentskills.io/home).

Each skill lives in its own directory under `skills/`, anchored by a `SKILL.md` file. Agentic tool (Such as Claude Code, Codex, etc.) loads a skill automatically when the task at hand matches its `description`.

## Available skills

| Skill | Description |
| --- | --- |
| [safety-deletion](skills/safety-deletion/) | Route every deletion inside a git-managed working tree through `git rm` / `git clean` instead of destructive commands (`rm`, `rmdir`, `shred`, …), keeping deletions recoverable through git history and the reflog. |

## Installation

Using the [skills CLI](https://github.com/obra/skills):

```bash
npx skills add knokmki612/safety-deletion
```

Or copy a skill directory manually into your skills directory:

```bash
# Enable for a single project
cp -r skills/safety-deletion .claude/skills/

# Enable for all projects (user-wide)
cp -r skills/safety-deletion ~/.claude/skills/
```

Once installed, Agentic tool reads the `description` from each `SKILL.md` and invokes the skill automatically when a relevant operation comes up.

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
