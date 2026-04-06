# Parameter Resolution

Parameters in workflow YAML are either **literal constants** or **`$`-expressions** that reference runtime values.

## Syntax

Values that start with `$` are resolved at runtime. Values without `$` are passed as-is.

```yaml
params:
  encounterId: $input.encounterId                              # from workflow input
  caseId: $activities.snapshot.output.billingCaseId           # from ancestor output
  resourceType: Patient                                        # literal string "Patient"
  status: active                                               # literal string "active"
```

## Expression types

### `$input.<field>`

References a field from the workflow's input object.

```yaml
input:
  encounterId: string
  tenantOrganizationId: string

activities:
  - id: snapshot
    params:
      encounterId: $input.encounterId
      tenantId: $input.tenantOrganizationId
```

### `$activities.<id>.output.<field>`

References a field from a completed ancestor activity's output. Supports dot notation for nested access.

```yaml
- id: fetch-context
  params:
    caseId: $activities.snapshot-billing-case.output.billingCaseId

- id: select-coverage
  params:
    context: $activities.fetch-context.output             # entire output object
    patient: $activities.fetch-context.output.patient     # nested field
```

## Ancestor-only rule

An activity can only reference **ancestors** in the tree — its parent, grandparent, and so on upward. Siblings and activities in other branches cannot be referenced.

This is a fundamental constraint: the tree structure encodes data dependencies. If activity B needs output from activity A, B must be a descendant of A.

```yaml
# CORRECT: build-claim is a descendant of fetch-context
activities:
  - id: fetch-context
    children: [build-claim]
  - id: build-claim
    params:
      context: $activities.fetch-context.output   # ✓ ancestor

# WRONG: two siblings cannot reference each other
  - id: step-a
    children: []
  - id: step-b
    params:
      x: $activities.step-a.output.value          # ✗ sibling — not resolved
```

{% hint style="info" %}
When a workflow needs data from two separate branches, use an **evaluator activity** that sits above both branches and produces the routing flags each branch needs. See [Conditional Execution](conditional-execution.md).
{% endhint %}

## Passing entire output objects

When an activity produces a complex object, pass the entire output to the downstream activity:

```yaml
- id: select-coverage
  params:
    context: $activities.fetch-context.output   # entire output as one param
```

The downstream TypeScript activity receives the full typed object.

## Missing fields

If a referenced field is `undefined` in the ancestor's output, the parameter receives `undefined`. Use `applicability` to guard against skipping an activity when a field is absent — see [Conditional Execution](conditional-execution.md).
