# Overview

RCMbox is a headless billing backend that sits between your clinical system and your payers. It runs your billing logic as configurable, durable workflows — producing X12 EDI claims, remittance postings, billing account transactions, or whatever your process requires.

## Architecture

RCMbox consists of four services:

| Service | Role |
|---------|------|
| **FHIR Server** | Stores clinical and billing data (Aidbox by default) |
| **Temporal** | Workflow engine — executes workflows durably with retries and history |
| **Billing API** | HTTP server — triggers workflows, manages the config project, serves the admin UI |
| **Billing Worker** | Temporal worker — loads workflow definitions and executes activity scripts |

## Config project

All client-specific logic lives in a **config project** — a separate git repository containing:

- `workflows/` — YAML workflow definitions
- `activities/` — project-specific TypeScript activity scripts
- `validation-rules/` — validation rule manifests and implementations
- `triggers/` — subscription and schedule trigger definitions

The Billing API clones and manages this repo. Each client gets their own branch, giving full isolation with a single shared engine.

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

RCMbox ships with a ready-made library of workflows for the claims lifecycle — pre-bill, claim submission, ERA processing, denial handling, and more — but you can define any workflow your process needs.

## What's next

{% content-ref %}
[Quick Start](quick-start.md)
{% endcontent-ref %}
