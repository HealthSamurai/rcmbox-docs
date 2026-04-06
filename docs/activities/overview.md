# Overview

An activity is a TypeScript file that implements one step of a workflow. Each activity exports a `main` function and a `description` string.

## Anatomy

```typescript
export const description = `
# Select Coverage

Selects the primary coverage to bill against.

## Input
- **context** — BillingCaseContext from fetch-context
- **coverageId** — (optional) explicit Coverage ID override

## Output
- **coverage** — the selected Coverage resource
- **selectionReason** — why this coverage was selected
- **hasCoverage** — true if a coverage was found
- **noCoverage** — true if no active coverage exists
`;

interface Input {
  context: BillingCaseContext;
  coverageId?: string;
}

interface Output {
  coverage?: Record<string, unknown>;
  selectionReason: string;
  hasCoverage: boolean;
  noCoverage: boolean;
}

export async function main(input: Input): Promise<Output> {
  // implementation
  const coverage = selectBestCoverage(input.context, input.coverageId);
  if (!coverage) {
    return { selectionReason: "no active coverage", hasCoverage: false, noCoverage: true };
  }
  return { coverage, selectionReason: "active PPO", hasCoverage: true, noCoverage: false };
}
```

## Required exports

| Export | Required | Description |
|---|---|---|
| `main` | yes | The activity function — called by the workflow engine |
| `description` | yes for UI | Markdown string shown in the admin UI activity browser |

## Two kinds of activities

| Kind | Location | YAML reference |
|---|---|---|
| Built-in | Shipped with the worker image | `@aidbox-billing/category/name` |
| Project-specific | In the config project | `./activities/workflow-id/name.ts` |

Built-in activities implement generic billing logic reusable across all clients. Project-specific activities implement client-specific decisions (coverage selection strategy, custom save logic, etc.).

{% content-ref %}
[Writing an Activity](writing-an-activity.md)
{% endcontent-ref %}

{% content-ref %}
[Built-In vs Project-Specific](built-in-vs-project-specific.md)
{% endcontent-ref %}

{% content-ref %}
[Importing Built-In Activities](importing-built-in-activities.md)
{% endcontent-ref %}
