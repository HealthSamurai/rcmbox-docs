# Session Resumption

The AI agent runs as a Temporal workflow, which means each chat turn is a separate invocation. The agent maintains context across turns within a session through session IDs.

## How it works

When you start a conversation with the agent, the API creates a session. Each subsequent message in the same chat passes the `sessionId`, allowing Claude Code to resume with the context of what was already discussed and changed.

## Starting a new session

If you close the agent chat and return later, you start a fresh session with no prior context. To continue where you left off, describe what was already done or reference the changes visible in the diff view.

## Limitations

- Session context is not persisted indefinitely — it lives for the duration of the Temporal workflow.
- Very long sessions may lose early context as the conversation grows. If the agent seems to forget earlier instructions, start a new session and re-state the key requirements.
