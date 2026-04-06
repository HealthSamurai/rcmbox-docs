# parse-x12

Parses a raw X12 EDI string into a structured object using the full spec-based X12 parser with XSD schema support. Automatically detects the transaction type from the GS-08 element (supports 835, 837P, 837I, 277, 999).

**Script path:** `@aidbox-billing/x12/parse-x12`

## Input

| Parameter | Type | Description |
|---|---|---|
| `rawX12` | `string` | Raw X12 file content as a string |

## Output

| Field | Type | Description |
|---|---|---|
| `parsed` | `ParsedX12` | Full parsed X12 structure with loops, segments, and composite elements |

## Supported transaction types

| GS-08 Code | Transaction |
|---|---|
| `835` | Healthcare claim payment / remittance advice (ERA) |
| `837P` | Professional healthcare claim |
| `837I` | Institutional healthcare claim |
| `277` | Healthcare claim status response |
| `999` | Functional acknowledgment |

## Usage in workflow YAML

```yaml
- id: parse-x12
  script: "@aidbox-billing/x12/parse-x12"
  params:
    rawX12: $input.rawX12
  children: [map-835]
```

## The ParsedX12 type

```typescript
import type { ParsedX12 } from "@aidbox-billing/built-in-activities/x12/_types";
```

The `ParsedX12` type is available from the config project's `manifests/types/` declarations for use in project-specific activity TypeScript files.

## See also

{% content-ref %}
[build-x12](build-x12.md)
{% endcontent-ref %}

{% content-ref %}
[map-835-to-claim-responses](map-835-to-claim-responses.md)
{% endcontent-ref %}

{% content-ref %}
[X12 and EDI Overview](../../x12/overview.md)
{% endcontent-ref %}
