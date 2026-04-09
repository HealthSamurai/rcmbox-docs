# search

Searches for FHIR resources on the connected Aidbox instance.

**Script path:** `@aidbox-billing/fhir/search`

## Input

| Parameter | Type | Description |
|---|---|---|
| `url` | `string` | FHIR search URL path (e.g. `"Patient"`, `"Claim"`, `"Encounter?status=finished"`) |
| `params` | object (optional) | Search parameters as key-value pairs. Values can be a string or an array of strings for repeated parameters. |

## Output

| Field | Type | Description |
|---|---|---|
| `bundle` | Bundle | FHIR `Bundle` of type `searchset` containing the matching resources |

## Usage in workflow YAML

```yaml
- id: find-claim
  script: "@aidbox-billing/fhir/search"
  params:
    url: Claim
    params:
      status: active
      patient: $activities.fetch-context.output.patient.id
  children: [process-claim]
```

## Repeated parameters

Pass an array of values to repeat a parameter:

```yaml
params:
  url: Patient
  params:
    _id: ["patient-1", "patient-2", "patient-3"]
```

## Importing in TypeScript

```typescript
import { main as fhirSearch } from "@aidbox-billing/built-in-activities/fhir/search";

const { bundle } = await fhirSearch({
  url: "Claim",
  params: { status: "active", _count: "100" },
});
```

## See also

