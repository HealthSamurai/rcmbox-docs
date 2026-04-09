# Temporal (Workflow Engine)

Temporal is the durable execution engine that powers RCMbox workflows. It handles retry logic, execution history, and deduplication so billing processes remain reliable even when individual steps fail or the system restarts.

## What Temporal provides

**Durable execution** — if the worker crashes mid-workflow, Temporal replays the workflow from the last completed activity. Activities that already succeeded are not re-run; only the in-progress or pending ones resume.

**Full history** — every workflow execution is recorded: inputs, outputs, activity durations, failures, and retries. This history is queryable from the API and surfaced in the admin UI's run history view.

**Deduplication** — each Temporal workflow has an ID. If you try to start a workflow with an ID that is already running, Temporal rejects the new request (or silently deduplicates it, depending on configuration). This prevents double-billing from duplicate trigger events.

**Retries** — individual activities can be configured with retry counts and timeouts. Temporal handles the retry scheduling with configurable backoff.

## How RCMbox uses Temporal

RCMbox uses two Temporal workflow types:

| Workflow | Purpose |
|---|---|
| `billingWorkflow` | Main billing execution — loads YAML, runs activities |
| `agentWorkflow` | AI agent editing session |

All billing workflows run on the **`billing` task queue**. The Billing Worker listens on this queue.

### Starting a workflow

The Billing API creates a Temporal workflow via the SDK when:
- A manual `POST /workflows/:id/run` is called
- A subscription trigger fires (webhook arrives at `POST /triggers/:id/webhook`)
- A schedule trigger fires (Temporal Schedule calls `billingWorkflow` directly)

### Workflow IDs

Each workflow run has a Temporal workflow ID. RCMbox constructs this ID from the trigger and resource context, enabling deduplication:

- Trigger runs: `{workflowId}-trigger-{triggerId}-{timestamp}` (always unique) or a custom ID from the `_workflowId` field in the trigger's input mapping.
- Manual runs: generated UUID unless `_workflowId` is specified in the request body.

{% hint style="info" %}
Set `_workflowId` in your trigger's `input-mapping.ts` to enable deduplication. For example, `process-order-enc-${encounter.id}` ensures only one prebill workflow runs per encounter, even if the trigger fires multiple times.
{% endhint %}

### Polling run status

The API reads Temporal's workflow history to return run status:

```http
GET /workflows/:id/runs/:runId
```

Returns the current status, activity execution details, and outputs — sourced from both Temporal history and the `BillingWorkflow` resource in Aidbox.

## Default timeouts and retries

The worker configures default Temporal activity options:

| Setting | Default |
|---|---|
| `scheduleToCloseTimeout` | `30s` |
| `retry.maximumAttempts` | `1` (no retries) |

Per-activity overrides in workflow YAML take precedence.

## Temporal address

Configure via the `TEMPORAL_ADDRESS` environment variable (default: `localhost:7233`).
