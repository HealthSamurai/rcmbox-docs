# Subscription Triggers

A subscription trigger starts a workflow whenever a FHIR resource changes in Aidbox. It uses Aidbox's topic-based subscription system — when the resource change matches the trigger criteria, Aidbox posts a webhook to the Billing API, which runs the input mapping and starts the workflow.

## Files

Each subscription trigger requires two files:

```
triggers/
  encounter-finished/
    definition.yaml      ← trigger configuration
    input-mapping.ts     ← maps the resource to workflow input
```

## definition.yaml

```yaml
type: subscription
workflow: prebill                       # workflow ID to start

subscriptionTopic:                      # AidboxSubscriptionTopic body
  resourceType: AidboxSubscriptionTopic
  url: "http://billing/triggers/encounter-finished"
  status: active
  trigger:
    - resource: Encounter
      supportedInteraction: [create, update]
      fhirPathCriteria: "status = 'finished'"

destination:                            # optional AidboxTopicDestination overrides
  content: full-resource                # full-resource (default), id-only, or empty
```

### Fields

| Field | Required | Description |
|---|---|---|
| `type` | no | `subscription` (default if omitted) |
| `workflow` | yes | ID of the workflow to start |
| `subscriptionTopic` | yes | Full `AidboxSubscriptionTopic` resource body |
| `destination` | no | Overrides for the auto-generated `AidboxTopicDestination` |

### `subscriptionTopic.trigger`

Each entry in `trigger` defines a FHIR resource to watch:

| Field | Description |
|---|---|
| `resource` | FHIR resource type to watch (e.g., `Encounter`, `Claim`) |
| `supportedInteraction` | Which events to react to: `create`, `update`, `delete` |
| `fhirPathCriteria` | Optional FHIRPath expression — only fire if the resource matches |

## input-mapping.ts

The input mapping is a TypeScript file that receives the changed FHIR resource and returns the workflow input:

```typescript
export const description = `# Encounter Finished

Starts the prebill workflow when an Encounter reaches finished status.`;

export async function main(input: { resource: Record<string, unknown> }) {
  const encounter = input.resource;
  const encounterId = encounter.id as string;
  const tenantOrgRef = (encounter.serviceProvider as any)?.reference || "";
  const tenantOrganizationId = tenantOrgRef.split("/")[1];

  return {
    _workflowId: `prebill-enc-${encounterId}`,   // deduplication key
    encounterId,
    tenantOrganizationId,
    submissionType: "original",
  };
}
```

The returned object becomes the workflow's input. The special `_workflowId` field controls deduplication — see [Deduplication](deduplication.md).

## What happens on sync

When you sync triggers (`POST /triggers/sync`), two Aidbox resources are created per subscription trigger:

1. **`AidboxSubscriptionTopic`** with ID `ts-trigger-{triggerId}` — defines what to watch.
2. **`AidboxTopicDestination`** with ID `ts-trigger-dest-{triggerId}` — webhook pointing to `{TS_API_BASE_URL}/triggers/{triggerId}/webhook`.

The destination uses the `webhook-at-least-once` delivery guarantee: Aidbox will retry delivery if the webhook endpoint doesn't return a 2xx response.

## Webhook endpoint

```http
POST /triggers/:triggerId/webhook
```

Receives the resource from Aidbox, runs `input-mapping.ts`, and starts the workflow on Temporal.
