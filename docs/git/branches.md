# Working with Branches

The config project uses branches to isolate changes — feature development, per-client customization, and AI agent editing sessions all happen on separate branches without affecting production.

## Creating a branch

```http
POST /repo/branches
{ "name": "feature/new-rule", "from": "main" }
```

This creates a new branch from the specified base and sets up a [git worktree](worktrees.md) for it automatically.

## Branch-aware execution

Every API endpoint accepts an optional `?branch=` parameter. When you run a workflow on a branch, the worker loads the YAML and activity scripts from that branch's worktree — not from the main clone. This means you can test changes in isolation before merging.

## Viewing changes

```http
GET /repo/diff?branch=feature/new-rule
```

Returns both uncommitted and committed changes relative to main. The admin UI uses this to show a diff view.

## Pushing

```http
POST /repo/push?branch=feature/new-rule
```

Pushes the branch to the remote. From there, the standard PR/review workflow applies.

## Typical workflow

1. Create a branch from the admin UI or API.
2. Edit activities — manually, or via the [AI Agent](../agent/overview.md).
3. Run workflows against the branch to test.
4. Review changes in the diff view.
5. Push and create a PR when ready.
6. Merge to main — the next `POST /repo/sync` picks up the changes.
