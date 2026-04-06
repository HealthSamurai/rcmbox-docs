# Directory Structure

The config project follows a fixed directory structure that the worker and API understand.

## Layout

```
config-project/
├── workflows/              # Workflow YAML definitions
│   ├── prebill.yaml
│   ├── claim-submission.yaml
│   └── link-claim-response.yaml
├── activities/             # Project-specific TypeScript activity scripts
│   ├── shared/             # Activities used by multiple workflows
│   │   └── fhir-helpers.ts
│   └── {workflow-id}/      # Activities scoped to one workflow
│       ├── select-coverage.ts
│       ├── resolve-charge-items.ts
│       └── save.ts
├── validation-rules/       # Validation rule manifest + scripts
│   ├── manifest.yaml
│   └── {rule-id}.ts
├── triggers/               # Trigger definitions
│   └── {trigger-id}/
│       ├── definition.yaml
│       └── input-mapping.ts   # subscription triggers only
├── manifests/              # Auto-generated type declarations (do not edit)
│   └── types/
│       ├── built-in-activities.ts
│       └── fhir/
│           └── _types.ts
└── package.json            # Optional npm dependencies
```

## workflows/

One YAML file per workflow. The filename must match the workflow's `id` field. For example, `prebill.yaml` must have `id: prebill` at the top.

## activities/

Activity TypeScript files organized into subdirectories:

- `shared/` — activities used by more than one workflow. When an activity that started in a workflow-specific folder is later needed by a second workflow, move it here.
- `{workflow-id}/` — activities used only by that workflow. Keeping them here makes it easy to understand which activities belong to which workflow.

## validation-rules/

- `manifest.yaml` — declares all validation rules with metadata (id, name, category, severity, script path).
- `{rule-id}.ts` — the TypeScript implementation of each rule. Each file exports `main(input): Promise<{ passed: boolean; message?: string }>`.

## triggers/

Each trigger is a subdirectory named after the trigger's ID:

- `definition.yaml` — the trigger configuration (type, workflow to start, subscription topic or schedule spec).
- `input-mapping.ts` — subscription triggers only. Maps the incoming FHIR resource event to workflow input parameters.

## manifests/types/

Auto-generated TypeScript type declarations synced from the product. These give activity scripts IDE autocomplete and type checking when importing built-in activities.

{% hint style="warning" %}
Do not edit files in `manifests/`. They are regenerated on `POST /repo/sync-built-in` and any manual changes will be overwritten.
{% endhint %}

## package.json

Optional. If the config project needs third-party npm packages (e.g., a date library, a specific EDI parser), add them here. The API runs `npm install` automatically when this file changes on sync. See [npm Dependencies](npm-dependencies.md).
