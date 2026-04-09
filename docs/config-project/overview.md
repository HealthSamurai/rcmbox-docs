# Config Project

The config project is a separate git repository that holds everything client-specific about a billing implementation. The Billing Worker reads from it at runtime — when a workflow runs, the worker loads the YAML and imports the activity scripts from the config project.

## What lives in the config project

- **Workflow definitions** — YAML files in `workflows/` that define the billing processes for this client.
- **Project-specific activities** — TypeScript files in `activities/` that implement client-specific logic (coverage selection, custom saves, transformation rules).
- **Validation rules** — manifest YAML and TypeScript scripts in `validation-rules/`.
- **Triggers** — subscription and schedule trigger definitions in `triggers/`.
- **Auto-generated type declarations** — in `manifests/types/`, synced from the product (do not edit).

## What does not live in the config project

Built-in activities (like `fetch-context`, `build-draft-claim`, `parse-x12`) ship with the Billing Worker Docker image. They are referenced in workflow YAML but their code is not in the config project.

## How the worker uses it

The worker mounts the config project as a Docker volume at `/data/repo`. On each activity execution, it resolves the script path relative to the current branch's directory and dynamically imports the TypeScript file.

For how code changes take effect without a worker restart, see [Dynamic Reload](../git/dynamic-reload.md).

## Example structure

```
my-config-project/
├── workflows/
│   ├── prebill.yaml
│   ├── claim-submission.yaml
│   └── link-claim-response.yaml
├── activities/
│   ├── shared/
│   │   └── fhir-helpers.ts
│   ├── prebill/
│   │   ├── select-coverage.ts
│   │   ├── resolve-charge-items.ts
│   │   └── save.ts
│   └── claim-submission/
│       └── transmit.ts
├── validation-rules/
│   ├── manifest.yaml
│   └── require-patient-dob.ts
├── triggers/
│   └── encounter-finished/
│       ├── definition.yaml
│       └── input-mapping.ts
└── manifests/
    └── types/
        ├── built-in-activities.ts
        └── fhir/
            └── _types.ts
```

## Git-based workflow

The config project is a standard git repository. The Billing API manages it at runtime — cloning, pulling, creating branches, and pushing changes. Each branch gets its own [git worktree](../git/worktrees.md), so multiple branches can be active simultaneously without interfering with each other.

This enables a development workflow where you edit activities on a feature branch, test them by running workflows against that branch, and merge when ready — all without restarting any services. See [Syncing the Config Repo](../git/syncing.md) and [Working with Branches](../git/branches.md) for details.

{% content-ref %}
[Directory Structure](directory-structure.md)
{% endcontent-ref %}

