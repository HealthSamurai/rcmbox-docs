# evaluate-validation-rules

Pure evaluation activity. Reads a YAML manifest of TypeScript validation rules, runs each rule against the `BillingCaseContext`, and returns `BillingTask` resources for failed rules. Does **not** persist anything — use `fhir/transaction` downstream to save tasks.

**Script path:** `@aidbox-billing/billing-case/evaluate-validation-rules`

## Input

| Parameter | Type | Description |
|---|---|---|
| `context` | `BillingCaseContext` | Billing case context to validate |
| `manifestPath` | `string` | Path to the manifest YAML, relative to `WORKFLOW_DIR` |
| `category` | `string` (optional) | Filter rules by this category value |
| `ruleParams` | object (optional) | Extra data passed to every rule's `main` function |

## Output

| Field | Type | Description |
|---|---|---|
| `taskBundle` | Bundle | FHIR transaction bundle with `BillingTask` POST entries for failed rules |
| `assignedDashboard` | `string` | Dashboard from the highest-priority failed rule (or `undefined` if all passed) |
| `assignedWorklist` | `string` | Worklist from the highest-priority failed rule (or `undefined` if all passed) |
| `passed` | `boolean` | `true` if zero `error`-severity rules failed |
| `summary` | object | `{ total, passed, failed, warnings }` counts |

## Usage in workflow YAML

```yaml
- id: evaluate-rules
  name: Evaluate Validation Rules
  script: "@aidbox-billing/billing-case/evaluate-validation-rules"
  params:
    context: $activities.build-draft-claim.output.context
    manifestPath: validation-rules/manifest.yaml
  children: [assert-no-errors]
```

## What counts as a pass

`passed` is `true` when no rules with `severity: error` failed. Rules with `severity: warning` or `severity: information` do not affect `passed`.

## Manifest path

The `manifestPath` is resolved relative to the config project's `WORKFLOW_DIR`. The standard path is `validation-rules/manifest.yaml`.

## See also

{% content-ref %}
[Validation Rules Overview](../../validation-rules/overview.md)
{% endcontent-ref %}

{% content-ref %}
[Manifest Format](../../validation-rules/manifest.md)
{% endcontent-ref %}
