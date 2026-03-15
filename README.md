# agent-skills

A collection of reusable AI agent behaviors, organized by domain. Designed to be pulled into your AI toolchain with [AIssemble](https://aissemble.io) or any compatible tool.

## Structure

Top-level directories are **domains** (e.g. `git`, `skill-creation`). Each domain can contain:

| Directory | Purpose |
|---|---|
| `skills/` | Skills — trigger-based instructions loaded into an AI assistant (e.g. Claude Code `SKILL.md` files) |
| `agents/` | Agent definitions — portable, reusable agent specs with I/O contracts, referencing a skill for behavior |
| `hooks/` | Hooks — scripts that run automatically in response to events (e.g. git hooks) |
| `instructions/` | Instructions — persistent rules loaded into an AI context (translated to Cursor rules, CLAUDE.md, etc.) |

### Skill vs Agent

**Skills** own the behavior. They define triggers, safety rules, and step-by-step workflows. They are the single source of truth for what gets done and how.

**Agents** are thin adapters. They declare a name, required tools, and input/output contract, then reference a skill for the actual logic. This makes the same behavior portable across different agent frameworks without duplicating it.

## Example

```
git/
├── skills/
│   └── update-branch/
│       └── SKILL.md          # full rebase workflow, subagent prompt
├── agents/
│   └── update-branch.md      # name, tools, I/O contract → points to skill
└── hooks/
    └── post-commit-rebase-check.md  # warns when feature branch falls behind mainline
```

## Usage

Pull a domain into your project with AIssemble, or copy individual files manually. Skills are loaded as context into your AI assistant; hooks are installed into `.git/hooks/` or your hook manager of choice.
