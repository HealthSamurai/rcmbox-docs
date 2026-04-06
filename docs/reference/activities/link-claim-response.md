# link-claim-response

Pure data transformation that links a matched `ClaimResponse` to its `Claim`. Sets the request reference, copies patient/insurer/requestor from the Claim, matches service lines by CPT code and modifiers, removes contained resources (replaced by external refs), and sets the match status to `matched`.

**Script path:** `@aidbox-billing/fhir/link-claim-response`

## Input

| Parameter | Type | Description |
|---|---|---|
| `claimResponse` | object | The draft `ClaimResponse` resource to link |
| `claim` | object | The matched `Claim` resource |

## Output

| Field | Type | Description |
|---|---|---|
| `bundle` | Bundle | FHIR transaction bundle with the updated (linked) `ClaimResponse` |

## What it does

1. Sets `ClaimResponse.request` to reference the matched Claim.
2. Copies `patient`, `insurer`, and `requestor` from the Claim (replacing contained resource refs with external refs).
3. Matches each `addItem` in the ClaimResponse to a `item` in the Claim by CPT code and modifiers.
4. Removes contained resources (since external refs now replace them).
5. Sets `status` to `active` and the `match-status` extension to `matched`.

## Usage in workflow YAML

```yaml
- id: link-match
  script: "@aidbox-billing/fhir/link-claim-response"
  applicability:
    - $activities.try-pcn-match.output.matched
  params:
    claimResponse: $activities.fetch.output.claimResponse
    claim: $activities.try-pcn-match.output.claim
  children: [save-match]
```

## See also

{% content-ref %}
[map-835-to-claim-responses](map-835-to-claim-responses.md)
{% endcontent-ref %}

{% content-ref %}
[Link ClaimResponse Workflow](../../workflows/link-claim-response.md)
{% endcontent-ref %}
