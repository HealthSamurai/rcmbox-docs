# Overview

RCMbox includes a general-purpose X12 EDI parser and builder, plus a set of built-in mappings between X12 transactions and FHIR resources. The parser and builder are generic — they work with any compliant X12 file. The mappings are provided as a working reference implementation that can be customized per client.

## Parser and builder

Two built-in activities handle raw X12:

| Activity | Purpose |
|---|---|
| `parse-x12` | Parses a raw X12 string into a structured `ParsedX12` object |
| `build-x12` | Converts a `ParsedX12` object back into a raw X12 string |

The parser auto-detects the transaction type from the GS-08 segment and supports **835**, **837P**, **837I**, **277**, and **999** formats. Parsing is spec-based (XSD schemas) and handles envelope extraction, composite elements, and separator detection automatically.

The builder reverses the process — it takes a `ParsedX12` structure and produces a valid X12 string with correct ISA fixed-length padding, HL ID sequencing, and SE segment counts.

Both activities are available as built-in references:
- `@aidbox-billing/x12/parse-x12`
- `@aidbox-billing/x12/build-x12`

## X12-to-FHIR mappings

RCMbox ships with built-in mapping activities for inbound X12 transactions:

| Activity | Direction | Description |
|---|---|---|
| `map-835-to-claim-responses` | X12 → FHIR | Maps 835 ERA remittance data to draft `ClaimResponse` resources with adjudication details, CARC/RARC codes, and service line breakdowns |
| `map-277-to-claim-responses` | X12 → FHIR | Maps 277 claim status responses to draft `ClaimResponse` resources with status codes and action indicators |

For outbound transactions, the FHIR-to-X12 mapping (e.g., `Claim` → 837P) is implemented as a project-specific activity in the config project, since it depends heavily on client-specific details like payer identifiers, provider addresses, and coding conventions.

{% hint style="info" %}
The built-in mappings are designed as a **reference implementation**. They cover the common structure of each transaction type and work out of the box for standard use cases. In practice, most clients customize these mappings — either by overriding the built-in activities with project-specific versions or by extending them to handle client-specific fields, identifier formats, and adjudication routing rules.
{% endhint %}

## What is typically customized

**Outbound (FHIR → X12)** — almost always client-specific:
- Provider and payer identifiers, names, and addresses
- Diagnosis code formatting and ordering
- Service line modifier mappings
- Resubmission and void handling logic

**Inbound (X12 → FHIR)** — often works out of the box, but may be customized for:
- Adjudication code routing (which CARC/RARC codes trigger which billing tasks)
- ClaimResponse enrichment with client-specific extensions
- Dashboard and worklist assignment rules

## The ParsedX12 intermediate

Both directions go through a `ParsedX12` intermediate structure rather than translating directly between raw X12 and FHIR. This decouples parsing from mapping:

```
Inbound:   raw X12 string → parse-x12 → ParsedX12 → mapping activity → FHIR resources
Outbound:  FHIR resources → mapping activity → ParsedX12 → build-x12 → raw X12 string
```

This means you can swap or customize the mapping layer without touching the parser or builder.
