# Link ClaimResponse

Matches a draft `ClaimResponse` (created by [Upload ERA](upload-era.md) or [Upload Claim Status](upload-claim-status.md)) to the original `Claim`. First attempts an exact match by Patient Control Number (PCN), then falls back to fuzzy matching by member ID. Marks the response as unmatched or ambiguous if no single match is found.

## Input

| Field | Type | Description |
|---|---|---|
| `claimResponseId` | string | FHIR ClaimResponse resource ID |
| `tenantOrganizationId` | string | Tenant Organization ID |

## Activity flow

fetch-claim-response → try-pcn-match → link-match | try-fuzzy-match → link-fuzzy-match | mark-unmatched | mark-ambiguous
