# Adjustment and Transfer

Processes a `BillingTask` that requires a contractual adjustment and balance transfer to the next payer. Creates three `BillingAccountTransaction` resources: an adjustment write-off, a transfer credit, and a transfer debit to the next responsible payer.

## Input

| Field | Type | Description |
|---|---|---|
| `taskId` | string | BillingTask resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-context → create-adjustment → create-transfer-credit → create-transfer-debit → prepare-save → save
