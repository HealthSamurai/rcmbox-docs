# Overview

Validation rules are TypeScript scripts that check a `BillingCaseContext` for billing errors before a claim is submitted. They live in the config project under `validation-rules/` and are evaluated by the `evaluate-validation-rules` built-in activity during the Prebill workflow.

## How validation works

The `evaluate-validation-rules` activity reads `validation-rules/manifest.yaml`, imports each rule's TypeScript script, and runs it against the current `BillingCaseContext`. Rules that fail produce `BillingTask` resources (persisted downstream) that appear in the billing worklist for human review.

A workflow that includes validation typically looks like this:

```yaml
- id: build-draft-claim
  children: [evaluate-rules]

- id: evaluate-rules
  script: "@aidbox-billing/billing-case/evaluate-validation-rules"
  params:
    context: $activities.build-draft-claim.output.context
    manifestPath: validation-rules/manifest.yaml
  children: [assert-no-errors]

- id: assert-no-errors
  script: ./activities/prebill/assert-no-errors.ts
  params:
    passed: $activities.evaluate-rules.output.passed
    summary: $activities.evaluate-rules.output.summary
    taskBundle: $activities.evaluate-rules.output.taskBundle
```

## Severity levels

| Severity | Effect |
|---|---|
| `error` | Counts as a failure — `passed` is `false` |
| `warning` | Recorded but does not fail — `passed` can still be `true` |
| `information` | Informational only |

The `assert-no-errors` activity (project-specific) decides what to do when `passed` is `false` — typically completing the workflow but marking the billing case as blocked for submission.

## What validation does NOT do

Validation is a **pure evaluation** — it does not write to Aidbox. The `evaluate-validation-rules` activity returns a `taskBundle` (a FHIR transaction bundle with `BillingTask` POST entries), which a downstream activity saves. This keeps validation side-effect-free and easy to test.

## Dashboard and worklist routing

Each rule in the manifest can specify `dashboard` and `worklist` metadata. When a rule fails, its BillingTask carries this metadata, routing the task to the correct billing worklist for human resolution.

{% content-ref %}
[Manifest Format](manifest.md)
{% endcontent-ref %}

{% content-ref %}
[Writing a Validation Rule](writing-a-rule.md)
{% endcontent-ref %}

{% content-ref %}
[Severity Levels](severity-levels.md)
{% endcontent-ref %}
