# Severity Levels

Each validation rule has a `severity` field that determines how a failure affects the workflow's `passed` output.

## Levels

| Severity | `passed` output | Description |
|---|---|---|
| `error` | `false` | The rule must pass for the claim to be submitted. |
| `warning` | `true` (still) | The rule failed, but the workflow can still proceed. |
| `information` | `true` (still) | Informational only — no action required. |

## How `passed` is calculated

The `evaluate-validation-rules` activity sets `passed: true` only when zero `error`-severity rules fail. Any number of `warning` or `information` failures can occur without affecting `passed`.

## What the billing team sees

All failures (error, warning, information) generate `BillingTask` resources that appear in the billing worklist. The severity is visible in the task, helping the team prioritize:

- **Error tasks** — must be resolved before submission can proceed.
- **Warning tasks** — should be reviewed, but submission is not blocked.
- **Information tasks** — for awareness only.

## Choosing a severity

Use `error` when the issue would cause a claim rejection by the payer (e.g., missing NPI, invalid date of birth format).

Use `warning` when the issue is worth reviewing but the claim might still be paid (e.g., missing secondary phone number, duplicate procedure code with distinguishing modifier).

Use `information` for compliance checks or internal QA notes that do not require action.

## Example manifest entries

```yaml
# Error — blocks submission
- id: demo-002-missing-dob
  severity: error
  message: "Patient must have a date of birth"

# Warning — shows in worklist but does not block
- id: demo-001-missing-patient-name
  severity: warning
  message: "Patient must have first and last name"

# Information — awareness only
- id: coverage-active-warning
  severity: warning
  message: "Patient should have active coverage"
```
