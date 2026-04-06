# Importing Built-In Activities

Project-specific activities can call built-in activities as TypeScript functions. This is the recommended pattern when you need to compose multiple built-in operations with custom logic in between.

## Setup

Type declarations for built-in activities are provided in `manifests/types/built-in-activities.ts`, which the config project's TypeScript compiler reads automatically. This gives you IDE autocomplete and type checking for all imports.

Keep these declarations up to date by syncing from the product:

```http
POST /repo/sync-built-in
```

Or use the "Sync Built-In Activities" button in the admin UI.

## Import syntax

```typescript
import { main as fhirSearch } from "@aidbox-billing/built-in-activities/fhir/search";
import { main as fhirTransaction } from "@aidbox-billing/built-in-activities/fhir/transaction";
import type { BillingCaseContext } from "@aidbox-billing/built-in-activities/billing-case/fetch-context";
```

## Example: save activity using built-ins

```typescript
import { main as fhirSearch } from "@aidbox-billing/built-in-activities/fhir/search";
import { main as fhirTransaction } from "@aidbox-billing/built-in-activities/fhir/transaction";
import type { Bundle } from "@aidbox-billing/built-in-activities/fhir/_types";

interface Input {
  bundle: Bundle;
  billingCaseId: string;
}

interface Output {
  saved: boolean;
}

export async function main(input: Input): Promise<Output> {
  // Check for existing record
  const { bundle: existing } = await fhirSearch({
    url: "BillingCase",
    params: { _id: input.billingCaseId },
  });

  if (existing.total === 0) {
    throw new Error(`BillingCase ${input.billingCaseId} not found`);
  }

  // Persist the transaction bundle
  await fhirTransaction({ bundle: input.bundle });

  return { saved: true };
}
```

## Available built-in activities

The full list with input/output types is in `manifests/types/built-in-activities.ts`. Key ones:

| Import path | What it does |
|---|---|
| `fhir/search` | Search for FHIR resources |
| `fhir/transaction` | Execute a FHIR transaction bundle |
| `fhir/link-claim-response` | Link a ClaimResponse to a Claim |
| `billing-case/fetch-context` | Fetch BillingCase context (also exports `BillingCaseContext` type) |
| `x12/parse-x12` | Parse raw X12 string |
| `x12/build-x12` | Build raw X12 string from parsed structure |

## Shared types

```typescript
import type { Bundle } from "@aidbox-billing/built-in-activities/fhir/_types";
import type { ParsedX12 } from "@aidbox-billing/built-in-activities/x12/_types";
```

These are available from `manifests/types/` — no separate import path needed.

## See also

{% content-ref %}
[Built-In vs Project-Specific](built-in-vs-project-specific.md)
{% endcontent-ref %}
