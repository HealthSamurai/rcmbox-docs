# Editing Workflows with the Agent

The agent understands the config project structure and can make targeted changes to workflows and activities through natural language prompts.

## Prerequisites

1. Create or select a feature branch in the admin UI — the agent is disabled on main.
2. Make sure the branch has a worktree (created automatically when you create a branch in the UI).

## Example prompts

**Adding a validation step:**
> After build-draft-claim, add a step that checks whether the patient's date of birth is present. If it's missing, log a warning but don't fail the workflow.

**Modifying coverage selection:**
> Update select-coverage.ts to prefer PPO plans over HMO plans when both are available. Fall back to HMO if no PPO exists.

**Adding a new activity:**
> Add an activity after save that sends a notification to Slack with the claim ID and total charge amount. The Slack webhook URL should come from the SLACK_WEBHOOK_URL environment variable.

**Understanding existing code:**
> Explain what the save-no-coverage activity does and when it runs.

## What the agent produces

For each change, the agent:
1. Reads relevant files to understand the current state.
2. Makes the minimum necessary edits.
3. Commits the changes with a descriptive commit message.

You can see the diff in the **Changes** tab in the admin UI after the agent completes.

## Testing changes

After the agent makes changes:
1. Go to the **Workflows** tab and select the modified workflow.
2. Use the **Run** button to execute it with a test input.
3. Inspect the activity outputs in the run history.
4. If something is wrong, ask the agent to fix it in a follow-up message.

## Iterating

The agent maintains session context — follow-up messages in the same session build on what was already done. If you close the session and return later, start a new session or describe the context again.

## Pushing to remote

Once satisfied with the changes:
1. Click **Push Branch** in the admin UI (or call `POST /repo/push?branch={name}`).
2. Open a pull request in your git hosting (GitHub, GitLab, etc.).
3. The pull request can be reviewed like any code change — the diff shows exactly what the agent modified.

{% content-ref %}
[Session Resumption](session-resumption.md)
{% endcontent-ref %}

{% content-ref %}
[Security and Isolation](security.md)
{% endcontent-ref %}
