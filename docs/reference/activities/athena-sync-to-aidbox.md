# athena/sync-to-aidbox

Syncs clinical data from Athena Health into Aidbox via bulk FHIR export. Pulls patient, encounter, procedure, practitioner, and coverage data from Athena's FHIR API, normalizes it, stages it in Azure Blob Storage, and triggers an Aidbox `$import` to load it.

**Script path:** `@aidbox-billing/athena/sync-to-aidbox`

## Input

| Parameter | Type | Description |
|---|---|---|
| `tenantOrganizationId` | `string` | Aidbox Organization ID scoping the sync |
| `since` | `string` (optional) | ISO date string — only sync resources updated since this date |

## Output

| Field | Type | Description |
|---|---|---|
| `importedCount` | `number` | Total number of FHIR resources imported |
| `resourceCounts` | object | Per-resource-type counts |
| `errors` | object[] | Any errors encountered during sync |

## Prerequisites

Configure the Athena credentials via environment variables:

| Variable | Description |
|---|---|
| `ATHENA_CLIENT_ID` | Athena API client ID |
| `ATHENA_SECRET` | Athena API client secret |
| `AZURE_STORAGE_ACCOUNT` | Azure Storage account name |
| `AZURE_CONTAINER` | Azure Blob container name |
| `AZURE_SAS_TOKEN` | Azure Shared Access Signature token |

## Usage in workflow YAML

```yaml
- id: sync-athena
  script: "@aidbox-billing/athena/sync-to-aidbox"
  timeout: "30m"
  params:
    tenantOrganizationId: $input.tenantOrganizationId
    since: $input.since
```

Use a long `timeout` — bulk exports can take several minutes depending on data volume.

## See also

{% content-ref %}
[cerner/sync-to-aidbox](cerner-sync-to-aidbox.md)
{% endcontent-ref %}

{% content-ref %}
[EHR Sync — Athena](../../workflows/ehr-sync-athena.md)
{% endcontent-ref %}
