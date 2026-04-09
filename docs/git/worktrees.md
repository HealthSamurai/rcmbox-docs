# Git Worktrees

The config project uses git worktrees to have multiple branches checked out simultaneously. Each branch gets its own directory on disk, sharing the same git object database as the main clone.

## Directory layout

```
/data/repo/                          # main branch (the primary clone)
/data/worktrees/feature-new-rule/    # feature branch worktree
/data/worktrees/agent-session-abc/   # AI agent working branch
```

## Why worktrees

Without worktrees, switching branches would require checking out a different commit in `/data/repo`, which would disrupt any in-flight workflows reading from that directory. Worktrees solve this by giving each branch its own filesystem path — the main branch stays untouched while feature branches are worked on in parallel.

## How they are managed

- **Created automatically** when a branch is created via `POST /repo/branches` or `POST /repo/worktrees`.
- **Shared `node_modules`** — each worktree gets a symlink to the main clone's `node_modules`, so npm dependencies don't need to be installed per branch.
- **Cleaned up** during `POST /repo/sync` — worktrees for branches that no longer exist on the remote are removed automatically.

## Worktrees and the worker

When a workflow runs on a branch, the Billing Worker resolves the config project directory to the branch's worktree path. Script imports use this path, so each branch runs its own version of the activity code.

Both the API and Worker containers must have access to the same `/data/worktrees` volume.
