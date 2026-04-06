# Schedule Triggers

A schedule trigger starts a workflow on a recurring schedule — an interval, a cron expression, or a calendar-based spec. It uses Temporal's native Schedule feature; no webhook is involved.

## Files

A schedule trigger only needs `definition.yaml` — no `input-mapping.ts`.

```
triggers/
  nightly-billing-report/
    definition.yaml
```

## definition.yaml

```yaml
type: schedule
workflow: generate-billing-report        # workflow ID to start

spec:                                    # Temporal ScheduleSpec
  intervals:
    - every: 24h

input:                                   # static workflow input
  reportType: daily
  tenantOrganizationId: org-1
```

### Fields

| Field | Required | Description |
|---|---|---|
| `type` | yes | Must be `schedule` |
| `workflow` | yes | ID of the workflow to start |
| `spec` | yes | Temporal ScheduleSpec — see below |
| `input` | no | Static input passed to the workflow on every run |

## Supported spec formats

The `spec` field maps directly to Temporal's `ScheduleSpec`.

### Interval

```yaml
spec:
  intervals:
    - every: 1h        # every hour
```

```yaml
spec:
  intervals:
    - every: 24h       # every 24 hours
    - every: 30m       # or every 30 minutes
```

### Cron expression

```yaml
spec:
  cronExpressions:
    - "0 2 * * *"      # 2:00 AM daily
```

### Calendar

```yaml
spec:
  calendars:
    - dayOfWeek: MONDAY
      hour: 6
      minute: 0
```

## What happens on sync

When you sync triggers (`POST /triggers/sync`), a Temporal Schedule is created with ID `ts-trigger-schedule-{triggerId}`. The schedule directly calls `billingWorkflow` with the static `input` on each firing.

If a schedule already exists with that ID, it is deleted and recreated (idempotent sync).

## Static input

The `input` field in `definition.yaml` is passed as-is to the workflow on every execution. Since there is no input mapping step, all values must be known at sync time — you cannot reference the triggering resource (there is none).

For dynamic input, use a [Subscription Trigger](subscription-triggers.md) instead.
