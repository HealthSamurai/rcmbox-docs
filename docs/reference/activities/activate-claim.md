# activate-claim

Takes a draft `Claim` and activates it for submission by setting its status to `active`. The draft already contains all snapshot data (contained resources, PCN identifier) from `build-draft-claim`, so activation is a single status flip.

**Script path:** `@aidbox-billing/billing-case/activate-claim`

## Input

| Parameter | Type | Description |
|---|---|---|
| `claimId` | `string` | ID of the draft `Claim` to activate |

## Output

| Field | Type | Description |
|---|---|---|
| `claim` | object | The activated `Claim` resource (status: `active`) |
| `chargeItems` | object[] | `ChargeItem` resources referenced by the claim |
| `billingCaseId` | `string` | ID of the `BillingCase` that references this claim |
| `bundle` | Bundle | FHIR transaction bundle with a PUT entry for the activated Claim |

## Usage in workflow YAML

```yaml
- id: activate-claim
  script: "@aidbox-billing/billing-case/activate-claim"
  params:
    claimId: $input.claimId
  children: [build-x12]
```

This activity is typically the first step in a Claim Submission workflow, which receives a `claimId` as input (the ID of a draft Claim that passed validation in the Prebill workflow).

## See also

{% content-ref %}
[build-draft-claim](build-draft-claim.md)
{% endcontent-ref %}

{% content-ref %}
[Claim Submission Workflow](../../workflows/claim-submission.md)
{% endcontent-ref %}
