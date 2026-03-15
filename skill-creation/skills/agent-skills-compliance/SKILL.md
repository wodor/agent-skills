---
name: agent-skills-compliance
description: Validate a skill directory or domain directory against the agentskills.io SKILL.md specification and local conventions. Reports precise PASS/FAIL/WARN entries and proposes minimal fixes. Use when creating, reviewing, or troubleshooting Agent Skills format compliance.
---

# Agent Skills Compliance

Validate skills and agent definitions against the Agent Skills specification at `https://agentskills.io/specification` and the conventions in this repository.

## Trigger

Use this skill when the user asks to:
- check if a skill or agent is spec-compliant
- validate `SKILL.md` or agent definition metadata
- troubleshoot skill loading or trigger issues
- lint or audit skill directories or a whole domain

## Inputs

- Target path: a skill directory, an agent file, or a domain directory (or multiple).
- If not provided, ask for a path.

## Validation Workflow

1. Determine the target type:
   - **Domain directory**: contains `skills/` and/or `agents/` subdirectories.
   - **Skill directory**: contains a `SKILL.md` file.
   - **Agent file**: a `.md` file inside an `agents/` directory.
2. Prefer automated validation first: `skills-ref validate <target>`
3. If `skills-ref` is unavailable, run manual checks below.
4. Return a structured report with `PASS`, `FAIL`, and `WARN` entries.
5. If there are failures, propose minimal edits and apply them if asked.

---

## Manual Checks — SKILL.md

### 1) Directory + required file

- `SKILL.md` must exist in the skill root.
- Skill directory must live under a `skills/` subdirectory of its domain (e.g. `git/skills/update-branch/`), not directly under the domain root.

### 2) Frontmatter presence and YAML validity

- `SKILL.md` must begin with YAML frontmatter delimited by `---`.
- Frontmatter must parse as valid YAML.

### 3) Required fields

- `name` is required.
- `description` is required.

### 4) `name` constraints

- Length: 1-64 chars.
- Allowed chars: lowercase letters, numbers, hyphens.
- Must not start or end with `-`.
- Must not contain `--`.
- Must exactly match the parent directory name.

### 5) `description` constraints

- Length: 1-1024 chars.
- Non-empty.
- Should clearly describe both what the skill does and when to use it.

### 6) Optional field constraints

- `compatibility` if present: max 500 chars.
- `metadata` if present: YAML mapping/object.
- `allowed-tools` if present: space-delimited string.

### 7) Body and structure recommendations (warn, not fail)

- Markdown body exists after frontmatter.
- Keep `SKILL.md` under ~500 lines.
- Use relative paths for internal references.
- Move large reference material into `references/` and keep core workflow in `SKILL.md`.
- If the skill delegates to a subagent, the subagent prompt should be inline in `SKILL.md` (not duplicated in the agent file) — the skill is the single source of truth for behavior.

---

## Manual Checks — Agent Definition

Agent definitions live at `<domain>/agents/<name>.md`.

### 1) File location

- Must be inside an `agents/` directory directly under the domain root (e.g. `git/agents/update-branch.md`).
- Filename (without `.md`) must match the `name` field in frontmatter.

### 2) Frontmatter presence and YAML validity

- Must begin with YAML frontmatter delimited by `---`.
- Frontmatter must parse as valid YAML.

### 3) Required fields

- `name`: same constraints as SKILL.md `name`.
- `description`: same constraints as SKILL.md `description`.
- `tools`: space-delimited string listing required tool capabilities (e.g. `shell`, `web`, `files`).

### 4) Optional fields

- `input`: YAML mapping of named input parameters, each with `description` and `required` (bool).
- `output`: YAML mapping of named output values with descriptions.
- `skill`: relative path to the `SKILL.md` that defines this agent's behavior.

### 5) Body recommendations (warn, not fail)

- Body should be short — a single paragraph or a few lines.
- If a `skill` reference is present, the body should defer to it rather than restating the workflow.
- Agent files must NOT duplicate step-by-step workflow that already lives in the referenced skill. Flag any body longer than ~20 lines as a `WARN`.

### 6) Skill reference integrity

- If `skill` is set, verify that the referenced file exists relative to the repo root.
- If it points to a `SKILL.md`, verify that file's `name` field is consistent with the agent's `name`.

---

## Domain Directory Checks (warn, not fail)

When the target is a domain directory:
- `skills/` subdirectory exists if any skills are present.
- `agents/` subdirectory exists if any agent definitions are present.
- No `SKILL.md` files sit directly inside the domain root (they belong under `skills/<name>/`).
- No agent `.md` files sit directly inside the domain root (they belong under `agents/`).

---

## Report Format

```text
## Validation Results

### /abs/path/to/target
- [PASS] SKILL.md exists
- [FAIL] name does not match directory (name: "x", dir: "y")
- [PASS] description length OK (142 chars)
- [WARN] SKILL.md is 620 lines; consider splitting into references/
- [WARN] agent body is 45 lines; workflow detail belongs in the skill, not the agent

## Suggested Fixes
1. Set frontmatter `name` to `skill-name`.
2. Move workflow steps from agent body into the referenced SKILL.md.
```

---

## Fix Strategy

When applying fixes, use the smallest safe edit set:
- Fix frontmatter fields first.
- Preserve user instruction body unless it conflicts with spec or conventions.
- When moving content from an agent file into a skill, verify the skill's subagent prompt section already exists or create it.
- Re-run validation after each fix batch.

## Notes

- Treat spec violations as `FAIL`.
- Treat best-practice and convention guidance as `WARN`.
- If requirements conflict across docs, prioritize `https://agentskills.io/specification`, then local conventions in this file.
