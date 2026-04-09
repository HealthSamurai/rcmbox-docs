# Process Billing Case ClaimResponses

Applies adjudication rules to newly linked `ClaimResponse` resources on a `BillingCase`. Creates `BillingTask` resources for items that require manual review (write-offs, transfers, denials) and updates the case's worklist category.

## Input

| Field | Type | Description |
|---|---|---|
| `billingCaseId` | string | BillingCase resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-case-and-new-crs → build-billing-tasks → prepare-save → save
