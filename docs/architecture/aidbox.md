# Aidbox (FHIR Server)

RCMbox is FHIR-native: all clinical and billing data — Patients, Encounters, Claims, CoverageEligibilityRequests, ChargeItems, and custom resources like `BillingCase` and `BillingTask` — are stored and exchanged as FHIR resources. RCMbox ships with [Aidbox](https://docs.aidbox.app) as its default FHIR server, but any FHIR R4-compliant server can be used in its place.

## Why Aidbox by default

Aidbox is a high-performance FHIR server built by Health Samurai. It offers features that the built-in activities rely on:

- **PostgreSQL-backed storage** — FHIR resources are stored in a Postgres schema, making complex queries and batch operations fast.
- **AidboxSubscriptionTopic** — the subscription trigger mechanism RCMbox uses to watch FHIR resource changes is implemented on top of Aidbox's topic-based subscription system.
- **Custom resource types** — `BillingCase`, `BillingTask`, `BillingTransmission`, and `BillingAccountTransaction` are registered as Aidbox custom resource definitions.
- **Contained resource support** — `BillingCase` snapshots clinical context as contained FHIR resources; Aidbox handles this transparently.

## Using a different FHIR server

If you bring your own FHIR server, consider the following:

| Feature | Aidbox | Any FHIR R4 server |
|---|---|---|
| Standard FHIR REST API | Yes | Yes |
| Subscription triggers | AidboxSubscriptionTopic | Requires custom trigger adapter |
| Custom resource types | Supported natively | May need profiles or extensions |
| Batch/transaction bundles | Full support | Required by FHIR spec (R4) |

Project-specific activities can call any FHIR server's REST API directly. Built-in activities that use Aidbox-specific features (subscriptions, custom resource types) may need adaptation if you switch servers.

## Connection

The FHIR server URL and credentials are configured via environment variables:

```
AIDBOX_URL=https://your-aidbox.example.com
AIDBOX_CLIENT_ID=rcmbox
AIDBOX_CLIENT_SECRET=...
```

Both the Billing API and Billing Worker use these to authenticate against the FHIR server.

## See also

{% content-ref %}
[System Architecture](overview.md)
{% endcontent-ref %}

{% content-ref %}
[BillingCase](../fhir/billing-case.md)
{% endcontent-ref %}
