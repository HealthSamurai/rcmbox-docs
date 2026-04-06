# Table of contents

* [RCMbox](README.md)

## Getting Started

* [Overview](getting-started/overview.md)
* [Quick Start](getting-started/quick-start.md)
* [Key Concepts](getting-started/key-concepts.md)

## Architecture

* [System Architecture](architecture/overview.md)
* [Aidbox (FHIR Server)](architecture/aidbox.md)
* [Temporal (Workflow Engine)](architecture/temporal.md)
* [Billing API](architecture/billing-api.md)
* [Billing Worker](architecture/billing-worker.md)
* [Multi-Tenancy](architecture/multi-tenancy.md)

## Config Project

* [Overview](config-project/overview.md)
* [Directory Structure](config-project/directory-structure.md)
* [Branch-per-Client Model](config-project/branching.md)
* [npm Dependencies](config-project/npm-dependencies.md)

## Workflow YAML

* [Overview](workflows/overview.md)
* [Activity Tree Structure](workflows/activity-tree.md)
* [Parameter Resolution](workflows/parameter-resolution.md)
* [Conditional Execution](workflows/conditional-execution.md)
* [Parallel Execution](workflows/parallel-execution.md)
* [Retries and Timeouts](workflows/retries-and-timeouts.md)

## Core RCM Workflows

* [Prebill](workflows/prebill.md)
* [Claim Submission](workflows/claim-submission.md)
* [Upload ERA (835)](workflows/upload-era.md)
* [Upload Claim Status (277)](workflows/upload-claim-status.md)
* [Link ClaimResponse](workflows/link-claim-response.md)
* [Process Billing Case ClaimResponses](workflows/process-billing-case-claim-responses.md)
* [Adjustment and Transfer](workflows/adjustment-and-transfer.md)
* [Contractual Write-Off](workflows/contractual-write-off.md)
* [Denial Auto and Transfer](workflows/denial-auto-and-transfer.md)
* [Create Billing Cases](workflows/create-billing-cases.md)
* [EHR Sync — Cerner](workflows/ehr-sync-cerner.md)
* [EHR Sync — Athena](workflows/ehr-sync-athena.md)

## Activities

* [Overview](activities/overview.md)
* [Writing an Activity](activities/writing-an-activity.md)
* [Built-In vs Project-Specific](activities/built-in-vs-project-specific.md)
* [Importing Built-In Activities](activities/importing-built-in-activities.md)

### Built-In Activity Reference

* [snapshot-billing-case](reference/activities/snapshot-billing-case.md)
* [fetch-context](reference/activities/fetch-context.md)
* [build-charge-items](reference/activities/build-charge-items.md)
* [build-draft-claim](reference/activities/build-draft-claim.md)
* [activate-claim](reference/activities/activate-claim.md)
* [evaluate-validation-rules](reference/activities/evaluate-validation-rules.md)
* [link-claim-response](reference/activities/link-claim-response.md)
* [map-835-to-claim-responses](reference/activities/map-835-to-claim-responses.md)
* [map-277-to-claim-responses](reference/activities/map-277-to-claim-responses.md)
* [parse-x12](reference/activities/parse-x12.md)
* [build-x12](reference/activities/build-x12.md)
* [search](reference/activities/search.md)
* [transaction](reference/activities/transaction.md)
* [athena/sync-to-aidbox](reference/activities/athena-sync-to-aidbox.md)
* [cerner/sync-to-aidbox](reference/activities/cerner-sync-to-aidbox.md)

## Triggers

* [Overview](triggers/overview.md)
* [Subscription Triggers](triggers/subscription-triggers.md)
* [Schedule Triggers](triggers/schedule-triggers.md)
* [Input Mapping](triggers/input-mapping.md)
* [Deduplication](triggers/deduplication.md)

## Validation Rules

* [Overview](validation-rules/overview.md)
* [Manifest Format](validation-rules/manifest.md)
* [Writing a Validation Rule](validation-rules/writing-a-rule.md)
* [Severity Levels](validation-rules/severity-levels.md)

## X12 and EDI

* [Overview](x12/overview.md)
* [837P — Professional Claims](x12/837p.md)
* [837I — Institutional Claims](x12/837i.md)
* [835 — Remittance Advice (ERA)](x12/835.md)
* [277 — Claim Status](x12/277.md)
* [999 — Functional Acknowledgment](x12/999.md)

## FHIR Data Model

* [Overview](fhir/overview.md)
* [BillingCase](fhir/billing-case.md)
* [ChargeItem and ChargeItemDefinition](fhir/charge-items.md)
* [Claim](fhir/claim.md)
* [ClaimResponse](fhir/claim-response.md)
* [Coverage](fhir/coverage.md)
* [BillingTransmission](fhir/billing-transmission.md)
* [BillingTask](fhir/billing-task.md)
* [BillingAccountTransaction](fhir/billing-account-transaction.md)
* [Patient Control Number (PCN)](fhir/pcn.md)

## Git and Version Control

* [Overview](git/overview.md)
* [Syncing the Config Repo](git/syncing.md)
* [Working with Branches](git/branches.md)
* [Git Worktrees](git/worktrees.md)
* [Dynamic Reload](git/dynamic-reload.md)
* [API Reference](git/api-reference.md)

## AI Agent

* [Overview](agent/overview.md)
* [Editing Workflows with the Agent](agent/editing-workflows.md)
* [Session Resumption](agent/session-resumption.md)
* [Security and Isolation](agent/security.md)
* [API Reference](agent/api-reference.md)
