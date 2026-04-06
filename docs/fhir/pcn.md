# Patient Control Number (PCN)

The Patient Control Number (PCN) is a unique identifier generated for each `Claim` resource. It is the primary key used to match an incoming `ClaimResponse` (from an 835 ERA or 277 file) back to the original claim.

## Where it lives

The PCN is stored in the `Claim.identifier` array:

```json
{
  "resourceType": "Claim",
  "identifier": [
    {
      "system": "http://aidbox.billing.com/CodeSystem/patient-control-number",
      "value": "PCN-a1b2c3d4"
    }
  ]
}
```

## When it is generated

The `build-draft-claim` activity generates the PCN at claim creation time and embeds it in the Claim. The PCN is preserved through all subsequent status changes (draft → active → response matched).

## How matching works

When an 835 ERA file is processed:

1. `parse-x12` reads the raw EDI.
2. `map-835-to-claim-responses` creates draft `ClaimResponse` resources. The PCN from the ERA's CLP segment is extracted into each `ClaimResponse.identifier`.
3. The link-claim-response workflow searches for a `Claim` whose PCN matches the `ClaimResponse`'s PCN.
4. If found: `link-claim-response` links the ClaimResponse to the Claim — match status: `matched`.
5. If not found: fuzzy matching by member ID is attempted; if that also fails, the ClaimResponse is marked `unmatched`.

## Match statuses

`ClaimResponse` resources carry a match-status extension:

| Status | Meaning |
|---|---|
| `matched` | Successfully linked to a Claim via PCN or fuzzy match |
| `unmatched` | No matching Claim found |
| `ambiguous` | Multiple potential Claims found — requires human review |

## Why PCN matters

Without a PCN, ERA matching relies entirely on fuzzy member ID matching, which is error-prone when a patient has multiple claims at a payer. PCN-based matching is exact and unambiguous. Clearinghouses pass the PCN back in the 835's CLP segment verbatim, making end-to-end claim tracking reliable.

## See also

{% content-ref %}
[Claim](claim.md)
{% endcontent-ref %}

{% content-ref %}
[ClaimResponse](claim-response.md)
{% endcontent-ref %}

{% content-ref %}
[map-835-to-claim-responses](../reference/activities/map-835-to-claim-responses.md)
{% endcontent-ref %}
