# Billing Worker

The Billing Worker is a Temporal worker that executes billing workflows. It listens on the `billing` task queue, loads workflow definitions from YAML, walks the activity tree, and dynamically imports and runs TypeScript activity scripts.

## How it works

1. The worker connects to Temporal and listens on the `billing` task queue.
2. When a workflow arrives, it creates a `BillingWorkflow` resource in Aidbox with status `in-progress`.
3. It loads the workflow YAML from the config project directory.
4. It walks the activity tree depth-first, executing each activity sequentially.
5. For each activity: resolves parameters from context, imports the script, calls `main(params)`, stores the output.
6. On completion, it patches the `BillingWorkflow` status to `completed` (or `failed`).

## Activity execution

### Script resolution

The worker resolves activity scripts based on their path format:

| Format | Example | Resolves to |
|---|---|---|
| Built-in | `@aidbox-billing/billing-case/fetch-context` | `{builtInDir}/billing-case/fetch-context.ts` |
| Project-specific | `./activities/prebill/save.ts` | `{workflowDir}/activities/prebill/save.ts` |

### Parameter resolution

Activity parameters are resolved from the workflow context:

| Syntax | Example | Resolves to |
|---|---|---|
| `$input.field` | `$input.encounterId` | Workflow input value |
| `$activities.id.output.field` | `$activities.fetch-context.output.coverages` | Output from an ancestor activity |
| Literal string | `"Patient"` | The string as-is |

Activities can only reference outputs from **ancestors** in the tree (parent, grandparent, etc.), not siblings.

### Applicability checks

Before executing an activity, the worker evaluates its `applicability` conditions. If any condition resolves to a falsy value, the activity and its children are skipped.

## Branch-aware execution

The worker resolves the config project directory based on how the workflow was started:

- **Main branch** — `/data/repo` (the main clone)
- **Feature branches** — `/data/worktrees/{branch-name}` (a git worktree)

Workflows on feature branches use that branch's activity code without affecting production.

## Dynamic reload

By default, Node.js ESM caches modules by file path. In production, activity scripts are cached after first import — changes require a worker restart.

With `DYNAMIC_RELOAD=true`, the worker creates **commit-specific checkouts** so each commit gets a unique filesystem path, bypassing the ESM cache:

1. On each `executeScript` call, reads the HEAD commit hash from the config project directory.
2. Creates a checkout at `/data/checkouts/{hash}` (via `git worktree add --detach`).
3. Symlinks `node_modules` from the source directory into the checkout.
4. Imports the activity script from the checkout path.

This means config project changes are picked up on the next workflow run without restarting the worker. Old checkouts are cleaned up during `POST /repo/sync`.

{% hint style="info" %}
`DYNAMIC_RELOAD` is intended for development and staging. In production, leave it off and redeploy to pick up config changes.
{% endhint %}

## Status tracking

The worker manages `BillingWorkflow` FHIR resources throughout execution:

| Event | Action |
|---|---|
| Workflow starts | `POST /fhir/BillingWorkflow` with `status: "in-progress"` |
| Workflow completes | `PATCH` status to `completed`, set `executionPeriod.end` |
| Workflow fails | `PATCH` status to `failed`, set `executionPeriod.end` |

The resource includes Temporal identifiers so the API can correlate FHIR resources with Temporal history:

```json
{
  "identifier": [
    { "system": "urn:temporal:workflow-id", "value": "prebill-enc-123" },
    { "system": "urn:temporal:run-id", "value": "abc-def-ghi" }
  ]
}
```

## Multi-tenant execution

When a workflow includes a `tenantOrganizationId`, the worker scopes all FHIR API calls to that organization. Aidbox URLs are prefixed with `/Organization/{id}`, and the tenant context is propagated to activities via `AsyncLocalStorage`. See [Multi-Tenancy](../fhir/multi-tenancy.md).

## Configuration

| Variable | Default | Purpose |
|---|---|---|
| `WORKFLOW_DIR` | `/data/repo` | Config project directory |
| `BUILT_IN_DIR` | `../built-in-activities` | Built-in activities path |
| `CHECKOUTS_DIR` | `/data/checkouts` | Commit-specific checkout storage |
| `DYNAMIC_RELOAD` | off | Enable dynamic reload for config changes |

Temporal and FHIR server settings are shared with the API — see [Temporal](temporal.md) and [Aidbox](aidbox.md).

## See also

{% content-ref %}
[System Architecture](overview.md)
{% endcontent-ref %}

{% content-ref %}
[Temporal (Workflow Engine)](temporal.md)
{% endcontent-ref %}

{% content-ref %}
[Dynamic Reload](../git/dynamic-reload.md)
{% endcontent-ref %}
