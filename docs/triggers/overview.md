# Overview

Triggers automatically start workflows in response to external events. They live in the config project under `triggers/` and must be synced to activate.

## Types

| Type | When it fires | Mechanism |
|---|---|---|
| **Subscription** | When a FHIR resource changes in Aidbox | Aidbox topic-based subscriptions (webhook) |
| **Schedule** | On a recurring cron or interval | Temporal Schedules |

## Directory structure

Each trigger is a subdirectory named after its ID:

```
triggers/
  encounter-finished/          # subscription trigger
    definition.yaml
    input-mapping.ts
  nightly-billing-report/      # schedule trigger
    definition.yaml
```

## Activating triggers

Triggers must be synced before they take effect. Use the **Sync Triggers** button in the admin UI or call:

```http
POST /triggers/sync
```

This creates the necessary Aidbox subscription resources (for subscription triggers) or Temporal Schedules (for schedule triggers).

## How a workflow starts

When a trigger fires:

1. **Subscription:** Aidbox sends a webhook to `POST /triggers/:triggerId/webhook` with the changed resource.
2. The API runs the trigger's `input-mapping.ts` to transform the resource into workflow input.
3. The API starts a `billingWorkflow` on Temporal with the mapped input.

For schedule triggers, Temporal itself starts the workflow directly — no webhook involved.

{% content-ref %}
[Subscription Triggers](subscription-triggers.md)
{% endcontent-ref %}

{% content-ref %}
[Schedule Triggers](schedule-triggers.md)
{% endcontent-ref %}

{% content-ref %}
[Input Mapping](input-mapping.md)
{% endcontent-ref %}

{% content-ref %}
[Deduplication](deduplication.md)
{% endcontent-ref %}
