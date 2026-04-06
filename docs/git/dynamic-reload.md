# Dynamic Reload

By default, Node.js ESM caches modules by their file path. This means that after you edit an activity script in the config project, the Billing Worker must be restarted to pick up the change.

**Dynamic Reload** (`DYNAMIC_RELOAD=true`) bypasses the ESM cache by creating **commit-specific checkouts** for each unique HEAD commit. Since each checkout has a unique path, the module is treated as a new import and the cache is never hit.

## How it works

When `DYNAMIC_RELOAD=true`, the worker does the following before importing each activity script:

1. Reads the HEAD commit hash from the config project directory (main clone or worktree).
2. Checks whether a checkout exists at `/data/checkouts/{hash}`.
3. If not, creates it via `git worktree add --detach /data/checkouts/{hash} {hash}`.
4. Symlinks `node_modules` from the source directory into the checkout (so npm deps are available).
5. Imports the activity script from `/data/checkouts/{hash}/activities/...`.

Since the path includes the commit hash, Node.js sees each commit's scripts as new modules and bypasses the cache.

## Directory layout

```
/data/repo/                            # main branch clone
/data/worktrees/{branch}/              # feature branch worktrees
/data/checkouts/{commit-hash}/         # commit-specific checkouts (DYNAMIC_RELOAD only)
```

## Development vs production

| Mode | `DYNAMIC_RELOAD` | Behavior |
|---|---|---|
| Development | `true` | Changes take effect on next run after committing — no restart needed |
| Production | `false` (default) | Scripts are cached on first import — changes require a worker restart (deploy) |

**Recommended:** enable `DYNAMIC_RELOAD=true` in development environments and feature branch testing. Keep it off in production for zero overhead.

## Update workflow with DYNAMIC_RELOAD enabled

1. Edit an activity script in the config project.
2. Commit the change: `git add -A && git commit -m "update select-coverage logic"`.
3. Sync the worker: `POST /repo/sync`.
4. Run the workflow — the worker picks up the new commit's activity automatically.

No restart needed.

## Cleanup

Old checkouts accumulate over time. They are automatically pruned on `POST /repo/sync` — the sync removes checkouts for commits that are no longer referenced.
