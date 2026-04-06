# Writing a Validation Rule

Each validation rule is a TypeScript file that exports a `main` function and a `description` string. Rules are pure functions — they receive context and return a pass/fail result, without writing to Aidbox.

## Interface

```typescript
export const description = `
# Require Patient Date of Birth

Validates that the patient has a date of birth (birthDate field).
`;

interface Input {
  subject: Record<string, unknown>;   // the FHIR resource at the rule's level
  context: BillingCaseContext;        // full billing case context
  params?: Record<string, unknown>;   // optional extra params from ruleParams
}

interface Output {
  passed: boolean;
  message?: string;    // optional detail shown in the task when failed
}

export async function main(input: Input): Promise<Output> {
  const patient = input.context.patient;
  if (!patient.birthDate) {
    return { passed: false, message: "Patient is missing birthDate" };
  }
  return { passed: true };
}
```

## The `subject` field

The `subject` in `input` is the FHIR resource at the rule's declared `level`:

| Level | `subject` contains |
|---|---|
| `patient` | The `Patient` resource |
| `encounter` | The `Encounter` resource |
| `procedure` | A single `Procedure` resource (rule runs once per procedure) |
| `claim` | The assembled `Claim` resource |

For `procedure`-level rules, `main` is called once for each procedure in the billing case.

## Accessing context

The full `BillingCaseContext` is always available via `input.context`:

```typescript
export async function main(input: Input) {
  const { patient, encounter, procedures, coverages } = input.context;
  // ...
}
```

## Returning failure details

When `passed: false`, include a `message` with additional detail. This message is stored in the `BillingTask` resource and shown to the billing team in the worklist.

## Registering the rule

After writing the file, add it to `validation-rules/manifest.yaml`:

```yaml
rules:
  - id: require-patient-dob
    name: Require Date of Birth
    category: pre-bill
    level: patient
    severity: error
    message: "Patient must have a date of birth"
    script: ./require-patient-dob.ts
    dashboard: PBF
    worklist: Demographic
```

The path in `script` is relative to the `validation-rules/` directory.

## See also

{% content-ref %}
[Manifest Format](manifest.md)
{% endcontent-ref %}

{% content-ref %}
[Severity Levels](severity-levels.md)
{% endcontent-ref %}
