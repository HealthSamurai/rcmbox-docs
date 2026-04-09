# Upload Claim Status (277)

Parses an X12 277 claim status response from a `BillingTransmission` resource, maps the status information to draft FHIR `ClaimResponse` resources, and marks the transmission as processed.

## Input

| Field | Type | Description |
|---|---|---|
| `billingTransmissionId` | string | BillingTransmission resource ID containing the 277 file |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-transmission → parse-x12 → map-to-claim-responses → prepare-save → save
