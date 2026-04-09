# BillingAccountTransaction

`BillingAccountTransaction` is a custom resource that records a financial transaction on the billing ledger — payments, adjustments, write-offs, and balance transfers.

## Purpose

When a BillingTask is processed (e.g., by the [Contractual Write-Off](../workflows/contractual-write-off.md) or [Adjustment and Transfer](../workflows/adjustment-and-transfer.md) workflow), the result is one or more BillingAccountTransactions that record the financial impact.

## Key fields

| Field | Description |
|---|---|
| `type` | `payment`, `adjustment`, `write-off`, `transfer-credit`, `transfer-debit` |
| `amount` | Transaction amount |
| `billingCase` | Reference to the parent BillingCase |
| `billingTask` | Reference to the BillingTask that created this transaction |
| `reasonCode` | CARC reason code (e.g., CO for contractual obligation) |
