# map-835-to-claim-responses

Parses 835 (ERA) transactions from a `ParsedX12` structure and produces draft FHIR `ClaimResponse` resources. Each CLP loop in the 835 becomes one `ClaimResponse` with contained patient/payer/payee, service-line adjudication entries (CARC codes), and remark codes (RARC codes).

**Script path:** `@aidbox-billing/billing-case/map-835-to-claim-responses`

## Input

| Parameter | Type | Description |
|---|---|---|
| `parsed` | `ParsedX12` | Parsed X12 structure from `parse-x12` |
| `billingTransmissionId` | `string` | ID of the `BillingTransmission` resource for this ERA file |

## Output

| Field | Type | Description |
|---|---|---|
| `claimResponses` | object[] | Array of draft `ClaimResponse` FHIR resources |
| `billingTransmission` | object | Updated `BillingTransmission` resource (status: `processed`) |

## What it does

For each 835 transaction set in the parsed file:
1. Extracts the BPR segment for payment metadata (amount, payment method, date).
2. Extracts the NM1 segments for payer and payee identification.
3. For each CLP loop (one per claim):
   - Creates a draft `ClaimResponse` with the claim status code.
   - Adds adjudication entries from SVC/CAS segments (CARC codes).
   - Adds remark codes from NTE/LQ segments (RARC codes).
   - Embeds patient and payer as contained resources.

The resulting `ClaimResponse` resources are in draft status and must be linked to their original Claims via the [link-claim-response](link-claim-response.md) activity.

## Usage in workflow YAML

```yaml
- id: map-835
  script: "@aidbox-billing/billing-case/map-835-to-claim-responses"
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
[link-claim-response](link-claim-response.md)
{% endcontent-ref %}

{% content-ref %}
[Upload ERA (835)](../../workflows/upload-era.md)
{% endcontent-ref %}
