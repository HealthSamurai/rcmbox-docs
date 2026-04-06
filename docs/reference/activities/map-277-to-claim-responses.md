# map-277-to-claim-responses

Parses 277 (Claim Status Response) transactions from a `ParsedX12` structure and produces draft FHIR `ClaimResponse` resources. Traverses the 277 HL loop hierarchy (2000A → 2000B → 2000C → 2000D → 2200D) and maps claim status codes (STC segments) to ClaimResponse extensions.

**Script path:** `@aidbox-billing/billing-case/map-277-to-claim-responses`

## Input

| Parameter | Type | Description |
|---|---|---|
| `parsed` | `ParsedX12` | Parsed X12 structure from `parse-x12` |
| `billingTransmissionId` | `string` | ID of the `BillingTransmission` resource for this 277 file |

## Output

| Field | Type | Description |
|---|---|---|
| `claimResponses` | object[] | Array of draft `ClaimResponse` FHIR resources |
| `billingTransmission` | object | Updated `BillingTransmission` resource (status: `processed`) |

## What it produces

For each claim in the 277 (2200D HL loop):
- A draft `ClaimResponse` with the claim status category and entity codes as extensions.
- Contained patient, payer, and provider resources extracted from the HL hierarchy.
- Service line `addItem` entries from SVC segments.
- Identifiers extracted from REF segments (patient control number, payer claim number).

## Usage in workflow YAML

```yaml
- id: map-277
  script: "@aidbox-billing/billing-case/map-277-to-claim-responses"
  params:
    parsed: $activities.parse-x12.output.parsed
    billingTransmissionId: $input.billingTransmissionId
  children: [save-responses]
```

## See also

{% content-ref %}
[parse-x12](parse-x12.md)
{% endcontent-ref %}

{% content-ref %}
[Upload Claim Status (277)](../../workflows/upload-claim-status.md)
{% endcontent-ref %}
