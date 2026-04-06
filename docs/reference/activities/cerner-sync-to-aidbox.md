# cerner/sync-to-aidbox

Syncs clinical data from Oracle Cerner into Aidbox via bulk FHIR export. Pulls patient, encounter, procedure, practitioner, and coverage data from Cerner's FHIR R4 API, normalizes it, stages it in Azure Blob Storage, and triggers an Aidbox `$import`.

**Script path:** `@aidbox-billing/cerner/sync-to-aidbox`

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

Configure the Cerner credentials via environment variables:

| Variable | Description |
|---|---|
| `CERNER_TENANT_ID` | Cerner tenant GUID |
| `CERNER_CLIENT_ID` | Cerner API client ID |
| `CERNER_CLIENT_SECRET` | Cerner API client secret |
| `AZURE_STORAGE_ACCOUNT` | Azure Storage account name |
| `AZURE_CONTAINER` | Azure Blob container name |
| `AZURE_SAS_TOKEN` | Azure Shared Access Signature token |

## Usage in workflow YAML

```yaml
- id: sync-cerner
  script: "@aidbox-billing/cerner/sync-to-aidbox"
  timeout: "30m"
  params:
    tenantOrganizationId: $input.tenantOrganizationId
    since: $input.since
```

## See also

{% content-ref %}
[athena/sync-to-aidbox](athena-sync-to-aidbox.md)
{% endcontent-ref %}

{% content-ref %}
[EHR Sync — Cerner](../../workflows/ehr-sync-cerner.md)
{% endcontent-ref %}
