# Default Policy

## Bad AI Code

```ts
const port = Number(process.env.PORT || 3000);
const timeoutMs = Number(process.env.REQUEST_TIMEOUT_MS || 5000);
const databaseUrl = process.env.DATABASE_URL || "postgres://localhost:5432/app";
const apiKey = process.env.API_KEY || "dev-key";
```

## What It Hides

This code treats every missing value as the same kind of problem. But defaults have different risk levels.

- `PORT=3000` is often fine.
- `REQUEST_TIMEOUT_MS=5000` may be fine after validation.
- `DATABASE_URL=localhost` can point production at the wrong database.
- `API_KEY=dev-key` is not a real credential.

## Better Semantics

Use a policy table before writing code:

| Variable | Policy |
| --- | --- |
| `PORT` | Optional with validated default |
| `REQUEST_TIMEOUT_MS` | Optional with validated default |
| `DATABASE_URL` | Required; no default |
| `API_KEY` | Required secret; no default |

Then implement the parser from that table.

## Review Questions

- Is the default low-risk?
- Is it valid in production?
- Does it hide deployment failure?
- Is it explicitly parsed and range-checked?
- Is dev-only behavior separated from production behavior?
