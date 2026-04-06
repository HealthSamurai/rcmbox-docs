# Retries and Timeouts

Each activity in a workflow can have its own retry count and timeout. These are configured per activity in the workflow YAML.

## Timeout

The `timeout` field sets how long the worker will wait for an activity to complete before failing it. Use a duration string.

```yaml
- id: transmit-claim
  timeout: "2m"
  script: ./activities/submission/transmit.ts
  params:
    claimId: $activities.activate.output.claimId
```

**Default:** `"30s"`

Supported units: `s` (seconds), `m` (minutes), `h` (hours). Examples: `"30s"`, `"5m"`, `"1h"`.

## Retries

The `retries` field sets how many times the worker will retry an activity after a failure before giving up.

```yaml
- id: call-clearinghouse
  retries: 3
  timeout: "1m"
  script: ./activities/submission/call-clearinghouse.ts
  params:
    payload: $activities.build-x12.output.rawX12
```

**Default:** `0` (no retries — the activity fails immediately on error).

Retries are handled by Temporal. Each retry attempt receives the same input. If all retries are exhausted, the workflow fails at that activity.

## Activity-level vs workflow-level defaults

There are no workflow-level defaults in the YAML — set `retries` and `timeout` per activity as needed. In practice:

- **Read-only activities** (FHIR search, context loading): low timeout, no retries needed.
- **External API calls** (clearinghouse, EHR APIs): higher timeout and 2–3 retries to handle transient failures.
- **Pure transforms** (build charge items, build claim): low timeout, no retries.
- **FHIR writes** (transaction bundles): moderate timeout, 0–1 retries.

## Example

```yaml
activities:
  - id: fetch-context
    script: "@aidbox-billing/billing-case/fetch-context"
    timeout: "30s"
    params:
      caseId: $activities.snapshot.output.billingCaseId

  - id: call-clearinghouse
    script: ./activities/submission/transmit.ts
    timeout: "2m"
    retries: 3
    params:
      rawX12: $activities.build-x12.output.rawX12

  - id: save
    script: ./activities/submission/save.ts
    timeout: "30s"
    params:
      bundle: $activities.build-x12.output.bundle
```
