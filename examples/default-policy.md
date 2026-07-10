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

- `PORT=3000` is only eligible if an existing deployment policy defines it.
- `REQUEST_TIMEOUT_MS=5000` is only eligible if an existing operations policy defines it and invalid values are rejected.
- `DATABASE_URL=localhost` can point production at the wrong database.
- `API_KEY=dev-key` is not a real credential.

Low risk is necessary for a default, but it is not policy evidence and does not authorize adding one.

## Better Semantics

Use repository or user evidence to build a policy table before writing code. This illustrative table assumes the first two policies are documented:

| Variable | Policy | Required evidence |
| --- | --- | --- |
| `PORT` | Preserve optional validated default | Deployment config or contract test defining `3000` |
| `REQUEST_TIMEOUT_MS` | Preserve optional validated default | Runbook, SLO, or contract test defining `5000` |
| `DATABASE_URL` | Required; remove default | Service config contract |
| `API_KEY` | Required secret; remove default | Integration config contract |

Then implement the parser from that table.

If evidence for a proposed default is absent, do not add it. If removing an existing default would change a public contract and the surrounding code cannot resolve the intent, mark `Needs policy` instead of guessing.

## Review Questions

- Is the default low-risk?
- What exact pre-existing evidence defines it?
- Is it valid in production?
- Does it hide deployment failure?
- Is it explicitly parsed and range-checked?
- Is dev-only behavior separated from production behavior?
