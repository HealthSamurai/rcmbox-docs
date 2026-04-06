# npm Dependencies

The config project can have its own `package.json` with npm dependencies. This lets project-specific activities import third-party packages without rebuilding the worker Docker image.

## Adding a dependency

Add a `package.json` to the root of the config project:

```json
{
  "name": "billing-config",
  "private": true,
  "type": "module",
  "dependencies": {
    "date-fns": "^3.6.0",
    "zod": "^3.23.0"
  }
}
```

Then use the package in any activity:

```typescript
import { format, differenceInDays } from "date-fns";

export async function main(input: Input): Promise<Output> {
  const dos = new Date(input.dateOfService);
  const today = new Date();
  const daysSince = differenceInDays(today, dos);
  return { age: daysSince, formatted: format(dos, "MM/dd/yyyy") };
}
```

## Automatic install

When `POST /repo/sync` is called and the sync detects that `package.json` changed (compared to the previous HEAD), the API automatically runs `npm install` in the config project directory before completing the sync.

You do not need to run `npm install` manually — syncing handles it.

## Branch worktrees and node_modules

Branch worktrees do not have their own `node_modules`. Instead, the API creates a symlink from `/data/worktrees/{branch}/node_modules` pointing to `/data/repo/node_modules`. This means all branches share the same installed packages.

If different branches need incompatible versions of a dependency, that is not currently supported — all branches share one `node_modules`. Design dependencies to be compatible across active branches.

## Scope

Config project npm dependencies are only available to project-specific activities (TypeScript files in `activities/` and `validation-rules/`). Built-in activities ship with their own bundled dependencies inside the worker Docker image.
