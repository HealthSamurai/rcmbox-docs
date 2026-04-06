# build-x12

Converts a parsed X12 structure back into a raw X12 string. Handles ISA fixed-length field padding, HL ID regeneration, and SE segment count calculation. Supports all transaction types produced by `parse-x12`.

**Script path:** `@aidbox-billing/x12/build-x12`

## Input

| Parameter | Type | Description |
|---|---|---|
| `parsed` | `ParsedX12` | Parsed X12 structure (as produced by `parse-x12`) |

## Output

| Field | Type | Description |
|---|---|---|
| `rawX12` | `string` | Raw X12 string ready for transmission |

## Usage in workflow YAML

```yaml
- id: build-x12
  script: "@aidbox-billing/x12/build-x12"
  params:
    parsed: $activities.map-to-x12.output.parsed
  children: [transmit]
```

This activity is the last step before transmission in claim submission workflows. A project-specific activity (`map-to-x12.ts`) typically constructs the `ParsedX12` structure from the FHIR Claim, and `build-x12` serializes it.

## See also

{% content-ref %}
[parse-x12](parse-x12.md)
{% endcontent-ref %}

{% content-ref %}
[Claim Submission Workflow](../../workflows/claim-submission.md)
{% endcontent-ref %}
