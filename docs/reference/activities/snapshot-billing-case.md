# snapshot-billing-case

Finds or creates a `BillingCase` for a given encounter. When creating, snapshots clinical and demographic data as contained resources for normalization. Idempotent — safe to call multiple times for the same encounter.

**Script path:** `@aidbox-billing/billing-case/snapshot-billing-case`

## Input

| Parameter | Type | Description |
|---|---|---|
| `encounterId` | `string` | ID of the FHIR `Encounter` resource to snapshot |

## Output

| Field | Type | Description |
|---|---|---|
| `billingCaseId` | `string` | ID of the found or created `BillingCase` |
| `created` | `boolean` | `true` if a new BillingCase was created, `false` if one already existed |

## What it does

1. Searches for an existing `BillingCase` linked to the encounter. If found, returns it immediately (idempotent).
2. If no BillingCase exists, fetches the encounter and then in parallel fetches the patient, active coverages, and procedures.
3. From procedures and encounter participants, extracts practitioner IDs. From coverage payors and encounter service provider, extracts organization IDs. From procedure reason references, extracts condition IDs.
4. Fetches all practitioners, organizations, and conditions in parallel.
5. Creates a `BillingCase` with all fetched resources embedded as contained resources, plus dual references (contained + external).

## Usage in workflow YAML

```yaml
- id: snapshot-billing-case
  name: Snapshot Billing Case
  script: "@aidbox-billing/billing-case/snapshot-billing-case"
  params:
    encounterId: $input.encounterId
  children: [fetch-context]
```

## See also

{% content-ref %}
[fetch-context](fetch-context.md)
{% endcontent-ref %}

{% content-ref %}
[Prebill Workflow](../../workflows/prebill.md)
{% endcontent-ref %}
