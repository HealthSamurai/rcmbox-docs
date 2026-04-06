# Built-In vs Project-Specific

RCMbox ships two kinds of activities: built-in activities that come with the product, and project-specific activities that the implementation team writes for each client.

## Comparison

| | Built-in | Project-specific |
|---|---|---|
| **Location** | `built-in-activities/` in the worker image | `activities/` in the config project |
| **Ships with** | Worker Docker image | Client deployment only |
| **YAML reference** | `@aidbox-billing/category/name` | `./activities/workflow-id/name.ts` |
| **Examples** | `fetch-context`, `build-draft-claim`, `parse-x12` | `select-coverage`, `save`, custom rules |
| **When to use** | Generic billing logic any client needs | Client-specific decisions and logic |

## Built-in activities

Built-in activities implement the core billing domain logic: snapshotting clinical data, building ChargeItems and Claims, parsing X12 files, executing FHIR operations. They are versioned with the product and available to all config projects.

Reference them in workflow YAML with the `@aidbox-billing/` prefix:

```yaml
- id: fetch-context
  script: "@aidbox-billing/billing-case/fetch-context"
  params:
    caseId: $activities.snapshot.output.billingCaseId
```

## Project-specific activities

Project-specific activities implement decisions that are specific to how a particular client operates — which coverage to pick, how to format a save bundle, custom transformation rules, integrations with client-specific systems.

Reference them with a relative path:

```yaml
- id: select-coverage
  script: ./activities/prebill/select-coverage.ts
  params:
    context: $activities.fetch-context.output
```

## Rule of thumb

If the activity would be useful for any RCMbox client, it belongs in built-in. If it encodes a specific client's business rule or preference, it belongs in the config project.

## Using built-in activities from project-specific code

Project-specific activities can import and call built-in activities as TypeScript functions:

```typescript
import { main as fhirSearch } from "@aidbox-billing/built-in-activities/fhir/search";
import { main as fhirTransaction } from "@aidbox-billing/built-in-activities/fhir/transaction";
```

This is useful when a project-specific activity needs to chain multiple FHIR operations with custom logic in between.

{% content-ref %}
[Importing Built-In Activities](importing-built-in-activities.md)
{% endcontent-ref %}
