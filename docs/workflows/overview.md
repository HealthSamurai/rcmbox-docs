# Overview

A workflow is a YAML file that describes a billing process as a tree of activities. The engine loads the YAML at runtime from the config project, walks the activity tree, and calls each activity's `main` function in order.

## Minimal example

```yaml
id: hello
name: Hello

input:
  patientId: string

rootChildren: [fetch-patient]

activities:
  - id: fetch-patient
    script: "@aidbox-billing/fhir/search"
    params:
      url: Patient
      params:
        _id: $input.patientId
    children: [save-result]

  - id: save-result
    script: ./activities/hello/save-result.ts
    params:
      bundle: $activities.fetch-patient.output.bundle
```

## Top-level fields

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique identifier. Must match the filename (without `.yaml`). |
| `name` | yes | Human-readable label shown in the UI. |
| `description` | no | Markdown description shown in the UI. |
| `input` | no | Map of `paramName: type` documenting expected input. |
| `rootChildren` | yes | List of activity IDs to execute first. |
| `activities` | yes | List of all activity definitions in the workflow. |

## Activity fields

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique within the workflow. Used to reference outputs via `$activities.id.output`. |
| `name` | no | Display name shown in the UI canvas. |
| `script` | yes* | Path to the activity script. `@aidbox-billing/...` for built-in, `./activities/...` for project-specific. |
| `params` | no | Map of parameter names to values (literal or `$`-expression). |
| `children` | no | IDs of activities to run after this one completes. |
| `applicability` | no | List of expressions — activity only runs if all are truthy. |
| `retries` | no | Number of retry attempts on failure (default: `0`). |
| `timeout` | no | Activity timeout as a duration string (e.g. `"30s"`, `"5m"`). Default: `"30s"`. |

\* `childWorkflow` can be used instead of `script` to invoke a sub-workflow by ID.

## Execution model

The engine walks the tree starting from `rootChildren`. For each activity:

1. Evaluate `applicability` — skip the activity (and its subtree) if any expression is falsy.
2. Resolve `params` — substitute `$input.*` and `$activities.*.output.*` expressions.
3. Import and call the activity's `main(params)` function.
4. Store the output as `activities[id].output`.
5. Run `children` in order.

Execution is sequential within a branch. When an activity has multiple children, those children form independent branches that run concurrently.

{% content-ref %}
[Activity Tree Structure](activity-tree.md)
{% endcontent-ref %}

{% content-ref %}
[Parameter Resolution](parameter-resolution.md)
{% endcontent-ref %}

{% content-ref %}
[Conditional Execution](conditional-execution.md)
{% endcontent-ref %}

{% content-ref %}
[Retries and Timeouts](retries-and-timeouts.md)
{% endcontent-ref %}
