# Input Mapping

The `input-mapping.ts` file in a subscription trigger transforms the incoming FHIR resource event into workflow input parameters. It is the bridge between the raw Aidbox webhook payload and the typed workflow input.

## Interface

Every `input-mapping.ts` exports:

- `main(input)` — **required** — the mapping function.
- `description` — **required for UI** — markdown description shown in the trigger list.

```typescript
export const description = `# Encounter Finished

Starts prebill when an Encounter status changes to "finished".`;

interface Input {
  resource: Record<string, unknown>;   // the FHIR resource from the webhook
}

export async function main(input: Input): Promise<Record<string, unknown>> {
  const encounter = input.resource;
  return {
    _workflowId: `prebill-enc-${encounter.id}`,
    encounterId: encounter.id as string,
    tenantOrganizationId: extractOrgId(encounter),
    submissionType: "original",
  };
}

function extractOrgId(encounter: Record<string, unknown>): string {
  const ref = (encounter.serviceProvider as { reference?: string })?.reference ?? "";
  return ref.split("/")[1] ?? "";
}
```

## The `_workflowId` field

The special `_workflowId` field controls the Temporal workflow ID for the started workflow. It is stripped from the input before the workflow receives it.

- **If provided:** used as the Temporal workflow ID. If a workflow with that ID is already running, Temporal rejects the new start request — this is the deduplication mechanism.
- **If omitted:** the ID is auto-generated as `{workflowId}-trigger-{triggerId}-{timestamp}`, which is always unique (no deduplication).

Set `_workflowId` to a resource-scoped value to ensure at most one workflow runs per resource:

```typescript
return {
  _workflowId: `prebill-enc-${encounter.id}`,   // one prebill per encounter
  encounterId: encounter.id as string,
};
```

{% content-ref %}
[Deduplication](deduplication.md)
{% endcontent-ref %}

## Resource content

By default, Aidbox delivers the full FHIR resource as the webhook payload. The `destination.content` field in `definition.yaml` controls this:

| Value | What arrives in `input.resource` |
|---|---|
| `full-resource` | Complete FHIR resource (default) |
| `id-only` | Resource with only `resourceType` and `id` |
| `empty` | Empty — you must fetch the resource yourself |

For `id-only` or `empty`, use `@aidbox-billing/fhir/search` inside `main` to fetch what you need.

## Async functions

`main` is `async` — you can make FHIR API calls inside it if you need to look up related resources before constructing the workflow input.
