---
name: post-commit-rebase-check
description: After each commit on a feature branch, periodically fetches from origin and warns if the mainline branch has moved ahead, making a rebase worthwhile.
git-hook: post-commit
---

# Hook: post-commit-rebase-check

Runs after every commit. Skips on mainline branches. Throttles the fetch to once every 10 minutes to avoid network overhead on every commit. Prints a warning when the feature branch has fallen behind mainline.

## Script

```sh
#!/usr/bin/env sh
set -e

# --- 1. Skip on mainline branches ---
BRANCH=$(git rev-parse --abbrev-ref HEAD)
for mainline in main master develop trunk; do
  if [ "$BRANCH" = "$mainline" ]; then
    exit 0
  fi
done

# --- 2. Throttled fetch (at most once every 10 minutes) ---
FETCH_STAMP="$(git rev-parse --git-dir)/post-commit-fetch-ts"
NOW=$(date +%s)
LAST=0
if [ -f "$FETCH_STAMP" ]; then
  LAST=$(cat "$FETCH_STAMP")
fi
ELAPSED=$(( NOW - LAST ))

if [ "$ELAPSED" -ge 600 ]; then
  git fetch --prune origin --quiet 2>/dev/null || true
  echo "$NOW" > "$FETCH_STAMP"
fi

# --- 3. Detect mainline branch from remote default, fallback to known names ---
MAINLINE=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's|origin/||') || true
if [ -z "$MAINLINE" ]; then
  for candidate in main master develop trunk; do
    if git show-ref --verify --quiet "refs/remotes/origin/$candidate"; then
      MAINLINE="$candidate"
      break
    fi
  done
fi

# No remote mainline found — nothing to compare against
[ -z "$MAINLINE" ] && exit 0

# --- 4. Count commits on mainline not in the feature branch ---
BEHIND=$(git rev-list --count "HEAD..origin/$MAINLINE" 2>/dev/null) || exit 0

if [ "$BEHIND" -gt 0 ]; then
  echo ""
  echo "  ⚠  '$BRANCH' is $BEHIND commit(s) behind origin/$MAINLINE."
  echo "     Consider rebasing: /update-branch"
  echo ""
fi
```

## Installation

Copy or symlink into the repository's hook directory:

```sh
cp post-commit-rebase-check.sh .git/hooks/post-commit
chmod +x .git/hooks/post-commit
```

Or with a hook manager (e.g. lefthook, husky, pre-commit):

```yaml
# lefthook.yml
post-commit:
  commands:
    rebase-check:
      run: sh path/to/post-commit-rebase-check.sh
```

## Tunables

| Variable / constant | Default | Description |
|---|---|---|
| `ELAPSED` threshold | `600` seconds | Minimum time between fetches. Lower for faster feedback, raise on slow networks. |
| Mainline candidates | `main master develop trunk` | Ordered fallback list when remote HEAD is not set. |

## Notes

- The fetch is best-effort (`|| true`) — a network failure silently skips the check rather than blocking the commit.
- The timestamp file lives inside `.git/` so it is never committed and is local to each clone.
- The warning message references `/update-branch` so the user can invoke the rebase skill directly.
