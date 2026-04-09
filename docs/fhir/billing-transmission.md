# BillingTransmission

`BillingTransmission` is a custom resource that represents an X12 file — either an outbound claim batch (837P) or an inbound response file (835 ERA, 277 status, 999 acknowledgment).

## Purpose

BillingTransmission stores the raw X12 content and tracks the processing status of the file. It acts as the input for the [Upload ERA](../workflows/upload-era.md) and [Upload Claim Status](../workflows/upload-claim-status.md) workflows.

## Key fields

| Field | Description |
|---|---|
| `status` | `pending` → `processed` (or `failed`) |
| `type` | Transaction type: `835`, `837p`, `277`, `999` |
| `content` | Raw X12 string |
| `claim` | References to Claims included in the transmission (outbound) |
| `claimResponse` | References to ClaimResponses created from this file (inbound) |
