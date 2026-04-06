# Conditional Execution

Use `applicability` to make an activity run only when certain conditions are met. If any condition is falsy, the activity and its entire subtree are skipped.

## Syntax

```yaml
- id: build-draft-claim
  applicability:
    - $activities.resolve-charge-items.output.hasChargeItems
  script: "@aidbox-billing/billing-case/build-draft-claim"
  params:
    chargeItems: $activities.resolve-charge-items.output.chargeItems
```

`applicability` is a list of `$`-expressions. The activity runs only when **all** expressions are truthy. Skipping an activity also skips all of its `children` recursively.

## The evaluator pattern

The most common use of `applicability` is to branch a workflow based on business logic. The recommended approach is an **evaluator activity** — a pure function that reads its parent's output and returns boolean routing flags:

```yaml
- id: try-pcn-match
  script: ./activities/link/try-pcn-match.ts
  params:
    claimResponse: $activities.fetch.output.claimResponse
    claims: $activities.fetch.output.claims
  children: [link-match, try-fuzzy-match]

- id: link-match
  applicability:
    - $activities.try-pcn-match.output.matched
  script: ./activities/link/link-match.ts
  params:
    claimResponse: $activities.fetch.output.claimResponse
    claim: $activities.try-pcn-match.output.claim

- id: try-fuzzy-match
  applicability:
    - $activities.try-pcn-match.output.noMatch
  script: ./activities/link/try-fuzzy-match.ts
  params:
    claimResponse: $activities.fetch.output.claimResponse
    claims: $activities.fetch.output.claims
```

Each branch's `applicability` checks the evaluator's output flags. Exactly one branch runs per execution.

## Real example: Prebill workflow

The actual prebill workflow uses multiple conditional branches:

```yaml
- id: select-coverage
  children: [resolve-charge-items, save-no-coverage]

- id: resolve-charge-items
  applicability:
    - $activities.select-coverage.output.hasCoverage
  children: [build-draft-claim, save-no-charges]

- id: build-draft-claim
  applicability:
    - $activities.resolve-charge-items.output.hasChargeItems
  children: [evaluate-rules]

- id: save-no-coverage
  applicability:
    - $activities.select-coverage.output.noCoverage

- id: save-no-charges
  applicability:
    - $activities.resolve-charge-items.output.noChargeItems
```

Each branch terminates with its own save activity — there is no shared merge point.

## Independent branch termination

Each branch should end with its own save/finalization step. Avoid creating a single activity that tries to merge outputs from multiple branches, since it would need sibling references — which are not allowed. See [Parameter Resolution](parameter-resolution.md).

## Decoupled activities

Set `decoupled: true` on an activity to run it independently — its failure does not affect the parent workflow. This is useful for side effects like sending notifications that should not block the main flow.

```yaml
- id: notify-team
  decoupled: true
  script: ./activities/shared/notify.ts
  params:
    message: $activities.save.output.status
```
