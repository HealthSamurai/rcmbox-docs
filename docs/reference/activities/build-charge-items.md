# build-charge-items

Fetches `ChargeItemDefinition` resources matching the procedure codes in the billing case, evaluates applicability expressions, and creates planned `ChargeItem` resources. Returns a FHIR transaction bundle for persistence.

**Script path:** `@aidbox-billing/billing-case/build-charge-items`

## Input

| Parameter | Type | Description |
|---|---|---|
| `context` | `BillingCaseContext` | Full billing case context from `fetch-context` |
| `coverage` | object | Selected Coverage resource — used for payer-specific filtering |
| `procedureCodeSystem` | `string` (optional) | Filter procedure codes by this coding system URL |

## Output

| Field | Type | Description |
|---|---|---|
| `chargeItems` | object[] | Created `ChargeItem` resources (status: `planned`) |
| `chargeItemDefinitions` | object[] | Matched `ChargeItemDefinition` resources |
| `bundle` | Bundle | FHIR transaction bundle with PUT entries for ChargeItems and the updated BillingCase |

Returns empty arrays and an empty bundle when no procedures match any definitions.

## Matching logic

For each procedure in the billing case context:

1. Extracts the procedure code (optionally filtered by `procedureCodeSystem`).
2. Fetches all `ChargeItemDefinition` resources matching that code via `context-type-value`.
3. Filters definitions by payer (via `useContext[program]` matching the coverage's payor organization IDs).
4. Filters by effective period (procedure/encounter date must fall within `effectivePeriod`).
5. Evaluates each definition's `applicability` JavaScript expressions against the procedure and context.
6. Creates one `ChargeItem` per matched (procedure, definition) pair.

## Usage in workflow YAML

```yaml
- id: build-charge-items
  script: "@aidbox-billing/billing-case/build-charge-items"
  params:
    context: $activities.fetch-context.output
    coverage: $activities.select-coverage.output.coverage
  children: [save]
```

## See also

{% content-ref %}
[build-draft-claim](build-draft-claim.md)
{% endcontent-ref %}

{% content-ref %}
[Prebill Workflow](../../workflows/prebill.md)
{% endcontent-ref %}
