# Deduplication

Triggers can fire multiple times for the same resource event — for example, if Aidbox retries a webhook delivery, or if a resource is updated several times in quick succession. Without deduplication, each firing would start a new workflow execution, potentially creating duplicate billing records.

## How it works

Deduplication is based on Temporal's workflow ID uniqueness guarantee. If you start a workflow with an ID that is already running, Temporal rejects the new request (raises `WorkflowExecutionAlreadyStartedError`). The Billing API swallows this error silently — the duplicate trigger is ignored.

## Setting up deduplication

In the trigger's `input-mapping.ts`, return a `_workflowId` field:

```typescript
export async function main(input: { resource: Record<string, unknown> }) {
  const encounter = input.resource;
  return {
    _workflowId: `prebill-enc-${encounter.id}`,   // ← deduplication key
    encounterId: encounter.id as string,
    tenantOrganizationId: "org-1",
  };
}
```

The `_workflowId` is stripped before the workflow receives its input.

**Rule:** if a workflow with this ID is already running, the new start is rejected. If no workflow is running with this ID, a new one starts normally.

## No deduplication (default)

If `_workflowId` is omitted, the ID is auto-generated as:

```
{workflowId}-trigger-{triggerId}-{timestamp}
```

This is always unique — every trigger firing starts a new workflow. Use this when you **want** multiple concurrent executions (e.g., processing each ERA file independently).

## After a workflow completes

Temporal's deduplication only applies to **running** workflows. Once a workflow completes or fails, its ID can be reused — a new execution with the same ID will start successfully.

This means deduplication prevents concurrent duplicates, not historical duplicates. If you need to prevent reprocessing a resource that was already processed, the workflow or activity logic itself must check for existing records (e.g., the `snapshot-billing-case` activity is idempotent — it returns an existing BillingCase rather than creating a new one).

## Manual runs

The same `_workflowId` mechanism works for manual runs via `POST /workflows/:id/run`:

```json
{
  "encounterId": "enc-123",
  "_workflowId": "prebill-enc-enc-123"
}
```

This allows the admin UI to manually trigger a workflow while still respecting deduplication.
