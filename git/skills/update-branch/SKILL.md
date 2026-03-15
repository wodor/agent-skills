---
name: update-branch
description: Rebase a feature branch onto the latest mainline branch commits (main, develop, or trunk). Use when the user wants to update their branch with the newest base branch changes while keeping history linear.
---

# Git Rebase Update Branch

Use this skill when the user is on a feature branch and wants to bring in the newest commits from the team's mainline branch via rebase.

## Trigger

Activate when the user asks to:
- rebase a feature branch on top of `main`, `develop`, or `trunk`
- update their branch with latest base-branch commits while preserving linear history
- sync branch with mainline without merge commits

## Execution

Delegate all work to a subagent to avoid consuming main-thread tokens. Use the Agent tool with `subagent_type: general-purpose` and the prompt below (substitute `<cwd>` with the actual working directory).

After the subagent completes, relay its summary to the user. If it reports that a force-push is needed, ask for explicit confirmation before running `git push --force-with-lease`.

### Subagent prompt

```
Working directory: <cwd>

Rebase the current feature branch onto the latest mainline commits by following the workflow below exactly.

## Safety constraints

- Never run `git reset --hard`, `git push --force`, or any other destructive command.
- If uncommitted changes are present, stash them (`git stash push -u`) before starting and restore them (`git stash pop`) when done.
- Never switch away from a branch without recording its name first.
- If any step produces an unexpected error, stop and include the raw error in your report.

## Steps

### 1. Verify repository state

```sh
git rev-parse --abbrev-ref HEAD   # must not return "HEAD" (detached)
git status --short
```

If HEAD is detached, stop and report: "Cannot rebase: repository is in detached HEAD state."

If there are uncommitted changes, stash them:

```sh
git stash push -u -m "update-branch: pre-rebase stash"
```

### 2. Identify the mainline branch

```sh
git symbolic-ref --short refs/remotes/origin/HEAD
```

If that fails, try `main`, `develop`, `trunk` in order and use the first one that exists locally.
If none exist, stop and report: "Cannot determine mainline branch. Please specify one."

### 3. Fetch from remote

```sh
git fetch --prune origin
```

### 4. Fast-forward the local mainline branch

```sh
git switch <mainline_branch>
git pull --ff-only origin <mainline_branch>
```

If `--ff-only` fails, stop and report the divergence. Do not proceed with the rebase.

### 5. Rebase the feature branch

```sh
git switch <feature_branch>
git rebase <mainline_branch>
```

### 6. Handle conflicts

For each conflict round:
1. `git status` — list conflicting files.
2. Resolve each file (remove all conflict markers, keep correct merged content).
3. `git add <resolved_file>` for each resolved file.
4. `git rebase --continue`

If the same files conflict repeatedly across more than three commits, abort:

```sh
git rebase --abort
```

Then report back with:
- How many commits were replayed before aborting.
- Which files conflicted repeatedly.
- Two options for the user to choose from:
  - **Squash + rebase**: squash feature commits interactively, then retry — fewer conflict rounds.
  - **Merge**: `git merge <mainline_branch>` — single conflict resolution, non-linear history.

### 7. Restore stash (if applicable)

```sh
git stash pop
```

### 8. Validate and report

```sh
git status
git log --oneline --decorate --graph -n 15
```

Report:
- Whether the rebase succeeded.
- Current branch and new base commit.
- Whether a force-push will be needed (true if the branch was previously pushed to remote).
- Any warnings.

Do NOT run `git push`.
```
