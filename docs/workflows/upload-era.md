# Upload ERA (835)

Parses an X12 835 remittance advice file from a `BillingTransmission` resource, maps the payment and adjustment data to draft FHIR `ClaimResponse` resources, and marks the transmission as processed.

## Input

| Field | Type | Description |
|---|---|---|
| `billingTransmissionId` | string | BillingTransmission resource ID containing the 835 file |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-transmission → parse-x12 → map-to-claim-responses → prepare-save → save
