# Syncing the Config Repo

The Billing API manages the config project's git state at runtime. The `POST /repo/sync` endpoint clones the repo on first call and pulls updates on subsequent calls.

## Initial clone

On startup (or first sync), the API clones the config project from `GIT_REPO_URL` into `/data/repo` and checks out the branch specified by `CONFIG_BRANCH`. If the repo already exists, it fetches and fast-forwards to the latest remote commit.

## What sync does

Each `POST /repo/sync` call:

1. **Fetches** the latest commits from the remote.
2. **Resets** the main branch to match the remote HEAD.
3. **Runs `npm install`** if `package.json` changed in the pull (so new config project dependencies are available immediately).
4. **Cleans up** stale worktrees whose remote branches no longer exist.
5. **Prunes** old dynamic reload checkouts that are no longer referenced.

## Auto-sync on startup

If `GIT_REPO_URL` is set, the API automatically syncs the config repo on boot before registering triggers or serving requests. This means a fresh deployment will have the latest config project state without manual intervention.

## URL rotation

If `GIT_REPO_URL` changes between restarts (e.g., migrating to a different repo), the API detects the mismatch, removes the old clone and all worktrees, and re-clones from the new URL.

## Configuration

| Variable | Purpose |
|---|---|
| `GIT_REPO_URL` | Remote repository URL |
| `GIT_TOKEN` | GitHub access token (used as `x-access-token` in the clone URL) |
| `CONFIG_BRANCH` | Branch to check out in `/data/repo` |
