# Security and Isolation

The AI agent is sandboxed to prevent unintended changes outside the config project.

## Filesystem isolation

The agent runs with `--cwd` set to the branch's git worktree directory (`/data/worktrees/{branch}`). It cannot read or write files outside this directory.

## Tool restrictions

The agent is limited to four tools:

| Tool | Purpose |
|---|---|
| `Read` | Read files in the worktree |
| `Write` | Create new files |
| `Edit` | Modify existing files |
| `Bash` | Run scoped git commands |

No network access, no FHIR API calls, no access to environment variables or secrets.

## Branch protection

The agent can only be activated on feature branches — the main branch is read-only in the admin UI. All changes go through the standard branch → PR → review → merge workflow.

## Concurrent sessions

Each branch has its own worktree, so multiple agent sessions on different branches run in complete isolation. Two agents editing different branches cannot interfere with each other.
