# Overview

RCMbox includes an AI agent that can edit workflows and activities directly in the config project through natural language. Billing engineers — and even non-technical billing ops staff — can describe what they need and the agent makes the changes, which can then be reviewed and tested before merging.

## What the agent can do

- Add, edit, or remove activities in a workflow YAML
- Write or modify project-specific activity TypeScript files
- Add or update validation rules
- Read and explain existing workflows and activities

## What the agent cannot do

- Edit the main branch directly (the UI enforces branch selection before enabling the agent)
- Run arbitrary shell commands (allowed tools are restricted to Read, Write, Edit, and scoped `git` commands)
- Access external systems — only the config project files in its worktree

## How it works

1. A user opens the agent chat in the admin UI and selects (or creates) a branch.
2. The user types a prompt describing what to change.
3. The API spawns Claude Code in headless mode, scoped to the branch's git worktree.
4. The agent reads existing files, makes changes, and commits them.
5. The user reviews the diff and runs the modified workflow to test it.
6. The user iterates (more chat turns, re-run, adjust) until satisfied.
7. The user pushes the branch and opens a pull request in the standard git workflow.

## Bring your own agent

RCMbox ships with Claude Code as the default agent, but the architecture is not locked to it. The agent integration is a thin layer — the API spawns a process scoped to a git worktree and streams the result. You can replace Claude Code with your own AI agent or coding assistant as long as it can operate on files in a given directory and commit changes to git.

## Isolation

The agent runs with `--cwd /data/worktrees/{branch}`, scoped entirely to that branch's directory. It cannot read or write files outside that worktree. The main branch is always read-only — the agent can only be activated on feature branches.

Each branch has its own worktree, so multiple agent sessions can run concurrently on different branches without interfering.

{% content-ref %}
[Editing Workflows with the Agent](editing-workflows.md)
{% endcontent-ref %}

{% content-ref %}
[Security and Isolation](security.md)
{% endcontent-ref %}
