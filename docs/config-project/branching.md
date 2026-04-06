# Branch-per-Client Model

RCMbox uses a **branch-per-client** model: the config project is a git repository, and different branches hold different versions of workflows and activities. The Billing Worker resolves the correct branch directory at runtime for each workflow execution.

## How branch resolution works

The worker determines which directory to load scripts from based on the `CONFIG_BRANCH` environment variable and the branch specified per workflow run:

| Source | Directory |
|---|---|
| `main` (default) | `/data/repo` — the main clone |
| Any other branch | `/data/worktrees/{branch-name}` — a git worktree |

When a workflow is triggered or run via the API, the caller can specify a `branch` parameter. If omitted, the worker uses the default `CONFIG_BRANCH`.

## Git worktrees

Branch directories other than `main` are **git worktrees** — lightweight checkouts that share the main repo's object database. This means:

- Multiple branches can be live simultaneously with no data duplication.
- Switching branches for a single workflow run does not affect other runs.
- Worktrees are created via the API (`POST /repo/branches` or `POST /repo/worktrees`) and cleaned up on `POST /repo/sync`.

```
/data/repo/                              # main branch
/data/worktrees/client-a-config/         # client A's branch
/data/worktrees/feature-new-rules/       # feature development branch
```

## Common branching patterns

### One branch per client

When the product hosts multiple clients, each client gets their own branch. Their workflows and activities are isolated — a change to `client-a`'s coverage selection logic does not affect `client-b`.

### Feature branches for development

When adding new rules or workflows, create a feature branch, test it against real data, then merge to the client's main branch via a standard pull request.

### Environment branches

Some implementations use branches for environments: `production`, `staging`, `development`. The worker in each environment points to the corresponding branch.

## Creating a new branch

Via the admin UI: use the **Branch Management** panel to create a new branch from any existing one.

Via the API:

```http
POST /repo/branches
{
  "branch": "client-b-config",
  "from": "main"
}
```

This creates the branch and its worktree in one step.

## Syncing

When the remote config project repo is updated (e.g., after a PR is merged), sync the worker:

```http
POST /repo/sync
```

This runs `git fetch --all --prune` on the main clone and updates all active worktrees. If `package.json` changed, `npm install` runs automatically.

{% content-ref %}
[Git Worktrees](../git/worktrees.md)
{% endcontent-ref %}

{% content-ref %}
[Dynamic Reload](../git/dynamic-reload.md)
{% endcontent-ref %}
