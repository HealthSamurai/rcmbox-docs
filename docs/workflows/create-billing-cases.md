# Create Billing Cases

Scheduled workflow that finds finished encounters without a billing case and creates draft cases for each. Designed to run on a schedule trigger (e.g., every 5 minutes) to automatically pick up new encounters.

## Input

No input — runs on a schedule trigger.

## Activity flow

fetch-tenants → find-and-create
