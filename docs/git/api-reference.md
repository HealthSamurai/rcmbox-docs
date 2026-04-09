# API Reference

Git operations on the config project are managed through the Billing API. All endpoints are available via Aidbox RPC.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/repo/sync` | Clone or pull the config repo, install deps, clean up stale worktrees |
| `GET` | `/repo/status` | Current commit hash, branch, and remote URL |
| `GET` | `/repo/branches` | List remote branches with commit hashes |
| `POST` | `/repo/branches` | Create a new branch with worktree |
| `POST` | `/repo/worktrees` | Create a worktree for an existing branch |
| `POST` | `/repo/push` | Push a branch to the remote |
| `GET` | `/repo/diff` | View uncommitted and committed changes vs main |
| `GET` | `/repo/commits` | Commit history for a branch |
| `POST` | `/repo/sync-built-in` | Sync built-in activity type declarations to the config project |

## Common parameters

| Parameter | Type | Description |
|---|---|---|
| `branch` | query | Branch name (defaults to main) |
| `name` | body | New branch name (for `POST /repo/branches`) |
| `from` | body | Base branch to create from (for `POST /repo/branches`) |
