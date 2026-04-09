# Multi-Tenancy

RCMbox supports multi-tenant deployments where a single instance serves multiple organizations — such as billing companies managing claims for several provider groups.

## How it works

Multi-tenancy is built on top of Aidbox Organization-Based Access Control (ORGBAC). Each tenant is represented as a FHIR `Organization` resource, and all clinical and billing data is scoped to the owning organization.

ORGBAC provides:

- **Hierarchical organization trees** — parent orgs can see data from child orgs
- **Role-based access** — users are assigned roles within specific organizations
- **Automatic data isolation** — queries only return resources belonging to the user's organization
- **Row-level security** — enforced at the database level, not just the API layer

## Data scoping

Every FHIR resource created in a multi-tenant deployment is tagged with the organization it belongs to. When a user queries the FHIR API, Aidbox automatically filters results to only include resources from organizations the user has access to.

This means workflows, triggers, and activities operate within the tenant boundary transparently — no special handling is needed in workflow YAML or activity code.