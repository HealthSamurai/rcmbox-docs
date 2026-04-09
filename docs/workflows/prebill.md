# Prebill

Creates a billing case, resolves charge items, builds a draft claim, and runs validation rules. Typically triggered when an encounter reaches `finished` status. Idempotent — safe to re-run (deletes old planned/draft resources before creating new ones).

## Input

| Field | Type | Description |
|---|---|---|
| `encounterId` | string | FHIR Encounter resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |
| `submissionType` | string | `"original"` \| `"replacement"` \| `"void"` (default: `"original"`) |
| `replacesClaimId` | string | ID of the previous Claim being replaced or voided |
| `coverageId` | string | Explicit Coverage ID override (for resubmissions) |

## Activity flow

snapshot-billing-case → fetch-context → select-coverage → resolve-charge-items → build-draft-claim → evaluate-rules → assert-no-errors → save

If no coverage is found or no charge items are resolved, the workflow exits early and persists the billing case with an appropriate status (`no-coverage` or `no-charges`).

## Outcome

On successful completion: a `BillingCase` linked to the encounter, `ChargeItem` resources for each matched procedure, a `Claim` in `draft` status with a PCN, and `BillingTask` resources for any validation warnings or errors.
