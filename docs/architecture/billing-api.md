# Billing API

The Billing API is an HTTP server (Fastify) that serves as the control plane for RCMbox. It triggers workflow execution, manages the config project via git, serves the admin UI, and handles inbound webhooks from subscription triggers.

## Responsibilities

| Area | What the API does |
|---|---|
| **Workflow runs** | Starts workflows via Temporal, returns run status and activity outputs |
| **Config project** | Clones the repo, creates branches and worktrees, pushes changes |
| **Triggers** | Receives Aidbox subscription webhooks, runs input mapping, starts workflows |
| **Activities** | Lists available activities (built-in and project-specific) with metadata |
| **Validation rules** | Lists validation rules from the config project manifest |
| **AI agent** | Spawns Claude Code sessions scoped to a branch worktree |

## Key endpoints

### Workflow execution

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/workflows/:id/run` | Trigger a workflow with input parameters |
| `GET` | `/workflows/:id/runs` | List all runs for a workflow |
| `GET` | `/workflows/:id/runs/:runId` | Get run details with activity outputs |
| `GET` | `/workflows` | List all workflow definitions |

### Config project management

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/repo/sync` | Clone or pull the config repo |
| `GET` | `/repo/status` | Current commit, branch, and remote URL |
| `GET` | `/repo/branches` | List remote branches |
| `POST` | `/repo/branches` | Create a new branch with worktree |
| `POST` | `/repo/worktrees` | Create worktree for existing branch |
| `POST` | `/repo/push` | Push a branch to remote |
| `GET` | `/repo/diff` | View changes vs main |
| `GET` | `/repo/commits` | Commit history for a branch |
| `POST` | `/repo/sync-built-in` | Sync built-in activity type declarations |

### Triggers and webhooks

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/triggers` | List all subscription and schedule triggers |
| `POST` | `/triggers/sync` | Register trigger definitions with Aidbox |
| `POST` | `/triggers/:triggerId/webhook` | Receive Aidbox subscription events |

### AI agent

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/agent/chat` | Start a Claude Code editing session |
| `GET` | `/agent/status/:workflowId` | Poll agent execution status |

## Branch-aware routing

Every endpoint accepts an optional `?branch=` query parameter. The API resolves the config project directory based on the branch:

- **Main branch** — reads from `/data/repo`
- **Feature branches** — reads from `/data/worktrees/{branch-name}`

This means the admin UI can browse workflows, activities, and validation rules from any branch without affecting production.

## Webhook flow

When an Aidbox subscription fires:

1. Aidbox sends a FHIR Bundle to `POST /triggers/:triggerId/webhook`
2. The API extracts the modified resources from the bundle
3. The trigger's `input-mapping.ts` script maps resources to workflow input parameters
4. The API starts a Temporal workflow on the `billing` task queue

## Startup tasks

On boot, the API:

1. Syncs the config repo from `GIT_REPO_URL` (if configured)
2. Registers itself as an Aidbox App (`/App/ts-billing-app`)
3. Uploads seed resources (AidboxQuery definitions, FAR packages)
4. Syncs trigger definitions from the config project

## Access control via Aidbox App

The Billing API is not exposed directly to clients. Instead, it registers itself as an [Aidbox App](aidbox.md) (`ts-billing-app`) on startup, and all endpoints are proxied through Aidbox's RPC protocol via `POST /rpc`. Each endpoint is registered as an Aidbox operation (e.g., `ts-billing-app.run-workflow`, `ts-billing-app.repo-sync`).

This means Aidbox manages all access control — authentication, authorization, and tenant scoping are handled by Aidbox's access policies before requests reach the Billing API. The admin UI and any external clients call Aidbox, never the API directly.

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `API_PORT` | `3000` | Server listen port |
| `WORKFLOW_DIR` | `/data/repo` | Config project directory |
| `BUILT_IN_DIR` | `../built-in-activities` | Built-in activities path |
| `WORKTREES_DIR` | `/data/worktrees` | Git worktree storage |
| `CONFIG_BRANCH` | (required) | Main branch name |
| `GIT_REPO_URL` | — | Config repo URL for auto-sync |
| `GIT_TOKEN` | — | GitHub access token |
| `TS_API_BASE_URL` | `http://host.docker.internal:3000` | Public URL for Aidbox App registration |

FHIR server and Temporal connection settings are shared with the worker — see [Aidbox](aidbox.md) and [Temporal](temporal.md).

## See also

{% content-ref %}
[System Architecture](overview.md)
{% endcontent-ref %}

{% content-ref %}
[Temporal (Workflow Engine)](temporal.md)
{% endcontent-ref %}

{% content-ref %}
[Subscription Triggers](../triggers/subscription-triggers.md)
{% endcontent-ref %}

{% content-ref %}
[AI Agent Overview](../agent/overview.md)
{% endcontent-ref %}
