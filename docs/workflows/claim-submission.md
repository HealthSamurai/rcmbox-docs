# Claim Submission

Takes a draft Claim produced by [Prebill](prebill.md), maps it to an X12 837P transaction, generates the raw X12 string, creates general ledger transactions, and persists everything atomically.

## Input

| Field | Type | Description |
|---|---|---|
| `claimId` | string | FHIR Claim resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

snapshot-claim → map-to-x12 → build-x12 → generate-gl → prepare-save → save
