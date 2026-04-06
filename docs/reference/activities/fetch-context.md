# fetch-context

Fetches a `BillingCase` and extracts its clinical context from contained resources. Claims, ChargeItems, and ClaimResponses are resolved via the FHIR API (external refs).

**Script path:** `@aidbox-billing/billing-case/fetch-context`

## Input

| Parameter | Type | Description |
|---|---|---|
| `caseId` | `string` | ID of the `BillingCase` to fetch context for |

## Output

Returns a `BillingCaseContext` object:

| Field | Type | Description |
|---|---|---|
| `billingCase` | object | The BillingCase resource |
| `patient` | object | Patient (from contained) |
| `encounter` | object | Primary Encounter (from contained) |
| `procedures` | object[] | Procedures (from contained) |
| `practitioners` | object[] | Practitioners (from contained) |
| `coverages` | object[] | Coverages (from contained) |
| `organizations` | object[] | Organizations (from contained) |
| `conditions` | object[] | Conditions (from contained) |
| `claims` | object[] | Claims referenced by the case (resolved via API) |
| `chargeItems` | object[] | ChargeItems referenced by the case (resolved via API) |
| `claimResponses` | object[] | ClaimResponses referenced by the case (resolved via API) |

## Usage in workflow YAML

```yaml
- id: fetch-context
  name: Fetch Case Context
  script: "@aidbox-billing/billing-case/fetch-context"
  params:
    caseId: $activities.snapshot-billing-case.output.billingCaseId
  children: [select-coverage]
```

## Importing in TypeScript

```typescript
import type { BillingCaseContext } from "@aidbox-billing/built-in-activities/billing-case/fetch-context";
```

The `BillingCaseContext` type is exported and used as the input type for downstream built-in activities like `build-charge-items` and `build-draft-claim`.

## See also

{% content-ref %}
[snapshot-billing-case](snapshot-billing-case.md)
{% endcontent-ref %}

{% content-ref %}
[build-charge-items](build-charge-items.md)
{% endcontent-ref %}
