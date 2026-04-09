# Dynamic Reload

By default, activity scripts are cached after first import — changes to the config project require a worker restart to take effect.

With `DYNAMIC_RELOAD=true`, the worker creates commit-specific checkouts so each commit gets a unique filesystem path, bypassing the cache. This means config project changes are picked up on the next workflow run without restarting the worker.

## Development vs production

| Mode | `DYNAMIC_RELOAD` | Behavior |
|---|---|---|
| Development | `true` | Changes take effect on next run after committing |
| Production | `false` (default) | Changes require a worker restart (deploy) |

## Update workflow

1. Edit an activity script in the config project.
2. Commit the change.
3. Sync the worker: `POST /repo/sync`.
4. Run the workflow — the worker picks up the new version automatically.

Old checkouts are pruned automatically during `POST /repo/sync`.
