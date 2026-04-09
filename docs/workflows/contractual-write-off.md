# Contractual Write-Off

Processes a `BillingTask` for a contractual write-off. Creates a single `BillingAccountTransaction` adjustment credit for the contractual obligation (CO reason code).

## Input

| Field | Type | Description |
|---|---|---|
| `taskId` | string | BillingTask resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-context → create-write-off → prepare-save → save
