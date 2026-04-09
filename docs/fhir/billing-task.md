# BillingTask

`BillingTask` is a custom resource representing a unit of work that requires action — typically generated after processing a ClaimResponse. Tasks drive the billing worklist and dashboard.

## Purpose

The [Process Billing Case ClaimResponses](../workflows/process-billing-case-claim-responses.md) workflow evaluates adjudication rules against incoming ClaimResponses and creates BillingTasks for items that need manual review or automated processing: contractual write-offs, balance transfers, denial handling, etc.

## Key fields

| Field | Description |
|---|---|
| `status` | `pending` → `completed` |
| `billingCase` | Reference to the parent BillingCase |
| `claimResponse` | Reference to the ClaimResponse that triggered this task |
| `category` | Task type (e.g., `contractual-write-off`, `adjustment-and-transfer`, `denial`) |
| `dashboard` | Which dashboard this task appears on |
| `worklist` | Which worklist category for manual review |

## Downstream workflows

BillingTasks are consumed by:

- [Contractual Write-Off](../workflows/contractual-write-off.md)
- [Adjustment and Transfer](../workflows/adjustment-and-transfer.md)
- [Denial Auto and Transfer](../workflows/denial-auto-and-transfer.md)
