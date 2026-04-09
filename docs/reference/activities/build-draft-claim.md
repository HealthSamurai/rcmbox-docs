# build-draft-claim

Takes `ChargeItem` resources, their `ChargeItemDefinition` pricing, and the `BillingCaseContext`. Generates a draft FHIR `Claim` with all service lines merged, clinical context embedded as contained resources, and a Patient Control Number (PCN) identifier for submission tracking.

**Script path:** `@aidbox-billing/billing-case/build-draft-claim`

## Input

| Parameter | Type | Description |
|---|---|---|
| `context` | `BillingCaseContext` | Billing case context from `fetch-context` |
| `coverage` | object | Selected Coverage resource |
| `chargeItems` | object[] | ChargeItem resources from `build-charge-items` |
| `chargeItemDefinitions` | object[] | Matched ChargeItemDefinitions from `build-charge-items` |
| `submissionType` | `string` (optional) | `"original"` \| `"replacement"` \| `"void"` — defaults to `"original"` |
| `replacesClaimId` | `string` (optional) | ID of the Claim being replaced or voided |

## Output

| Field | Type | Description |
|---|---|---|
| `context` | `BillingCaseContext` | Augmented context with `chargeItems` and `claims` populated |
| `claim` | object | The merged draft `Claim` resource (status: `draft`) |
| `bundle` | Bundle | FHIR transaction bundle with PUT entries for the Claim and updated BillingCase |

## What it does

1. Generates a per-ChargeItem Claim with pricing from the matching ChargeItemDefinition.
2. Merges all service lines into a single Claim resource.
3. Embeds the clinical context (patient, encounter, practitioners, organizations, coverages) as contained resources.
4. Generates a Patient Control Number (PCN) in the `identifier` array for claim matching downstream.
5. Sets `use` based on `submissionType`: original → `claim`, replacement → `preauthorization` with `related.type.code = "prior"`, void → `preauthorization` with `related.type.code = "prior"` and `related.relationship.code = "deceased"`.

## Usage in workflow YAML

```yaml
- id: build-draft-claim
  script: "@aidbox-billing/billing-case/build-draft-claim"
  applicability:
    - $activities.resolve-charge-items.output.hasChargeItems
  params:
    context: $activities.fetch-context.output
    coverage: $activities.select-coverage.output.coverage
    chargeItems: $activities.resolve-charge-items.output.chargeItems
    chargeItemDefinitions: $activities.resolve-charge-items.output.chargeItemDefinitions
    submissionType: $input.submissionType
    replacesClaimId: $input.replacesClaimId
  children: [evaluate-rules]
```

