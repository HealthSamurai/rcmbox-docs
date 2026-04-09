# RCMbox

RCMbox is a headless billing backend that sits between your clinical system and your payers. It provides:

- **[Workflow engine](architecture/overview.md) with [triggers](triggers/overview.md)** — a durable execution engine on top of Temporal that runs billing workflows defined in YAML, triggered by FHIR resource changes, schedules, or API calls.
- **[Prebuilt RCM workflows](workflows/prebill.md)** — ready-to-use workflows covering the full claims lifecycle: prebill, claim submission, ERA processing, claim status handling, denial management, and balance transfers. All workflows are configurable per client via a git-based [config project](config-project/overview.md).
- **[RCM data model based on FHIR](fhir/overview.md)** — standard FHIR resources (Claim, ClaimResponse, Coverage, ChargeItem) extended with custom resources (BillingCase, BillingTask, BillingTransmission, BillingAccountTransaction) that track the full billing lifecycle.
- **[X12 EDI tooling](x12/overview.md)** — a generic X12 parser and builder with reference mappings between X12 transactions (837P, 835, 277) and FHIR resources, customizable per client.
- **[AI agent](agent/overview.md)** — an integrated AI assistant that can edit workflows and activities through natural language, with support for bringing your own agent.

## Architecture

RCMbox consists of four services:

| Service | Role |
|---------|------|
| **FHIR Server** | Stores clinical and billing data (Aidbox by default) |
| **Temporal** | Workflow engine — executes workflows durably with retries and history |
| **Billing API** | HTTP server — triggers workflows, manages the config project, serves the admin UI |
| **Billing Worker** | Temporal worker — loads workflow definitions and executes activity scripts |

{% content-ref %}
[System Architecture](architecture/overview.md)
{% endcontent-ref %}

## Config project

All client-specific logic lives in a **config project** — a separate git repository containing:

- `workflows/` — YAML workflow definitions
- `activities/` — project-specific TypeScript activity scripts
- `validation-rules/` — validation rule manifests and implementations
- `triggers/` — subscription and schedule trigger definitions

The Billing API clones and manages this repo. Each client gets their own branch, giving full isolation with a single shared engine.

{% content-ref %}
[Config Project](config-project/overview.md)
{% endcontent-ref %}

## How a workflow runs

{% stepper %}
{% step %}
A trigger fires — a FHIR resource change, a schedule, or a direct API call — starting a workflow.
{% endstep %}
{% step %}
The Billing Worker loads the workflow YAML from the config project and begins executing activities in order.
{% endstep %}
{% step %}
Each activity reads its inputs from the workflow context (prior activity outputs or trigger input), runs its TypeScript logic, and returns an output.
{% endstep %}
{% step %}
The workflow completes. All inputs, outputs, and intermediate states are recorded in Temporal's execution history.
{% endstep %}
{% endstepper %}
