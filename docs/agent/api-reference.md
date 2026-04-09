# API Reference

The AI agent is controlled through two Billing API endpoints, available via Aidbox RPC.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/agent/chat` | Start or continue an agent session |
| `GET` | `/agent/status/:workflowId` | Poll execution status |

## Starting a session

```http
POST /agent/chat
{
  "branch": "feature/new-rule",
  "prompt": "Add a validation rule that checks for missing NPI",
  "sessionId": null
}
```

Returns a `workflowId` for polling and a `sessionId` for follow-up messages.

## Continuing a session

Pass the `sessionId` from the previous response to maintain conversation context:

```http
POST /agent/chat
{
  "branch": "feature/new-rule",
  "prompt": "Also add it to the prebill workflow after evaluate-rules",
  "sessionId": "session-abc-123"
}
```

## Polling status

```http
GET /agent/status/workflow-id-123
```

Returns the current execution state (`running`, `completed`, `failed`) and the agent's output when finished.
