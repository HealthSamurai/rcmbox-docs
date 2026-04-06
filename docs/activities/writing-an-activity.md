# Writing an Activity

An activity is a TypeScript file with two required exports: `description` and `main`.

## Minimal template

```typescript
export const description = `
# My Activity Name

One sentence explaining what this activity does.

## Input
- **param** — description of the param

## Output
- **result** — description of the output field
`;

interface Input {
  param: string;
}

interface Output {
  result: string;
}

export async function main(input: Input): Promise<Output> {
  // implementation
  return { result: `done: ${input.param}` };
}
```

## Rules

- `main` must be `async` and return a `Promise`.
- `description` must be a markdown string — it is shown in the admin UI's activity browser.
- The `Input` interface shapes what the workflow engine passes to `main` (from the workflow YAML `params`).
- The `Output` interface shapes what downstream activities can reference via `$activities.{id}.output.*`.

## Where to put the file

Project-specific activities go in the config project:

- `activities/{workflow-id}/{name}.ts` — for activities used by only one workflow.
- `activities/shared/{name}.ts` — for activities reused across multiple workflows.

## Referencing in workflow YAML

```yaml
- id: my-activity
  script: ./activities/prebill/my-activity.ts
  params:
    param: $input.myParam
```

## Using FHIR operations

Import built-in activities to read and write FHIR resources:

```typescript
import { main as fhirSearch } from "@aidbox-billing/built-in-activities/fhir/search";
import { main as fhirTransaction } from "@aidbox-billing/built-in-activities/fhir/transaction";

export async function main(input: Input): Promise<Output> {
  const { bundle } = await fhirSearch({ url: "Claim", params: { status: "active" } });
  // ... process results ...
  await fhirTransaction({ bundle: txBundle });
  return { status: "saved" };
}
```

## Side effects

Activities that write to Aidbox should do so at the end, in a single FHIR transaction. This makes them easier to test and reason about. Build the transaction bundle in the activity body, then call `fhirTransaction` at the very end.

## Error handling

Throw an `Error` to fail the activity. The workflow engine catches it, marks the activity as failed, and applies the configured retry policy. Include enough context in the error message for debugging:

```typescript
if (!coverage) {
  throw new Error(`No active coverage found for patient ${patientId}`);
}
```

## See also

{% content-ref %}
[Built-In vs Project-Specific](built-in-vs-project-specific.md)
{% endcontent-ref %}

{% content-ref %}
[Importing Built-In Activities](importing-built-in-activities.md)
{% endcontent-ref %}
