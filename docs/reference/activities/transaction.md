# transaction

Executes a FHIR transaction or batch bundle against the connected Aidbox instance. All entries in a transaction bundle are applied atomically — if any entry fails, the entire transaction is rolled back.

**Script path:** `@aidbox-billing/fhir/transaction`

## Input

| Parameter | Type | Description |
|---|---|---|
| `bundle` | Bundle | A FHIR `Bundle` with `type: "transaction"` or `type: "batch"`, containing entries with `request` (method, url) and `resource` |

## Output

| Field | Type | Description |
|---|---|---|
| `response` | Bundle | FHIR Bundle response with per-entry outcomes |

## Usage in workflow YAML

```yaml
- id: save
  script: "@aidbox-billing/fhir/transaction"
  params:
    bundle: $activities.build-claim.output.bundle
```

## Importing in TypeScript

```typescript
import { main as fhirTransaction } from "@aidbox-billing/built-in-activities/fhir/transaction";

const { response } = await fhirTransaction({ bundle: myBundle });
```

This is the standard way for project-specific save activities to persist resources. Build the transaction bundle in earlier activities (e.g., `build-charge-items`, `build-draft-claim`), then pass the bundle to a save step that calls `fhirTransaction`.

## Transaction vs batch

- `transaction`: atomic — all entries succeed or all fail.
- `batch`: non-atomic — entries are processed independently; some may fail while others succeed.

Use `transaction` for billing writes (creating ChargeItems + updating BillingCase together) to maintain data consistency.

## See also

{% content-ref %}
[search](search.md)
{% endcontent-ref %}
