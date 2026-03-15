---
name: update-branch
description: Rebases the current feature branch onto the latest mainline commits (main, develop, or trunk). Fetches from remote, fast-forwards the local mainline branch, rebases the feature branch, and resolves conflicts where possible.
tools: shell
input:
  cwd:
    description: Absolute path of the git repository to operate in.
    required: true
  mainline_branch:
    description: Branch to rebase onto. Auto-detected from remote if omitted.
    required: false
output:
  summary: Human-readable outcome (success, conflicts encountered, action needed).
  force_push_needed: Whether the branch needs a force-push after rebasing.
skill: git/skills/update-branch/SKILL.md
---

# Agent: update-branch

Follow the workflow defined in the referenced skill (`skill` field in frontmatter).
Inputs and outputs are declared above; all behavioral details, safety constraints, and step-by-step instructions are in the skill file.
