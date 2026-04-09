# Denial Auto and Transfer

Processes an auto-generated denial `BillingTask`. Creates three `BillingAccountTransaction` resources: a denial write-off, a transfer credit, and a transfer debit to the next responsible payer.

## Input

| Field | Type | Description |
|---|---|---|
| `taskId` | string | BillingTask resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-context → create-denial → create-transfer-credit → create-transfer-debit → prepare-save → save
