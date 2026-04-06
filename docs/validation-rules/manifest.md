# Manifest Format

The validation rules manifest is a YAML file at `validation-rules/manifest.yaml` that declares all rules for the config project. The `evaluate-validation-rules` activity reads this file at runtime to discover and run rules.

## Structure

```yaml
rules:
  - id: demo-002-missing-dob
    name: Missing Date of Birth
    category: pre-bill
    level: patient
    severity: error
    message: "Patient must have a date of birth"
    script: ./require-patient-dob.ts
    dashboard: PBF
    worklist: Demographic
```

## Rule fields

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique rule identifier |
| `name` | yes | Human-readable rule name shown in the UI |
| `category` | yes | Rule category — used to filter rules via the `category` param |
| `level` | yes | What the rule evaluates: `patient`, `encounter`, `procedure`, or `claim` |
| `severity` | yes | `error`, `warning`, or `information` |
| `message` | yes | User-facing message when the rule fails |
| `script` | yes | Path to the rule's TypeScript file, relative to `validation-rules/` |
| `dashboard` | no | Dashboard code for routing failed tasks (e.g., `PBF`) |
| `worklist` | no | Worklist name for routing failed tasks (e.g., `Demographic`) |

## Categories

Group rules by the workflow step they apply to. The `category` param in `evaluate-validation-rules` lets you run only a subset:

```yaml
- id: evaluate-rules
  script: "@aidbox-billing/billing-case/evaluate-validation-rules"
  params:
    context: $activities.build-draft-claim.output.context
    manifestPath: validation-rules/manifest.yaml
    category: pre-bill
```

## Severity levels

| Severity | Effect on `passed` |
|---|---|
| `error` | `passed: false` |
| `warning` | does not fail — `passed` can still be `true` |
| `information` | does not fail |

## Common worklists in the default manifest

| Worklist | Rules it covers |
|---|---|
| `Demographic` | Patient name, DOB, address, phone, subscriber ID |
| `Charge Entry` | Date of service, CPT codes, units, charge amounts |
| `Coding` | Diagnosis codes, DX pointers, modifiers |
| `Provider` | Rendering provider NPI |
| `Insurance` | Payer, plan type, coverage status |
| `Facility` | Admit/discharge dates |
| `EDI/Claim Structure` | DX pointer references, structural validity |
| `Pricing` | ChargeItem count, total charge |

## See also

{% content-ref %}
[Writing a Validation Rule](writing-a-rule.md)
{% endcontent-ref %}

{% content-ref %}
[Severity Levels](severity-levels.md)
{% endcontent-ref %}
