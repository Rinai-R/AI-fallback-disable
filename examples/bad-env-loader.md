# Bad Env Loader

## Bad AI Code

```ts
const config = {
  port: Number(process.env.PORT?.trim() || 3000),
  apiKey: process.env.API_KEY?.trim() || "",
  databaseUrl: process.env.DATABASE_URL?.trim() || "localhost",
  jwtSecret: process.env.JWT_SECRET?.trim() || "secret",
  timeoutMs: Number(process.env.TIMEOUT || 5000),
};
```

## What It Hides

- `apiKey` becomes an empty string instead of failing startup.
- `databaseUrl` may point to a local or wrong database.
- `jwtSecret` silently uses a weak default.
- `PORT` and `TIMEOUT` do not distinguish missing, blank, malformed, and valid values.
- All variables are treated the same even though they have different risk levels.

## Better Semantics

Read env once, classify each variable, parse by type, and export typed config.

```ts
export const config = loadConfig(process.env);
```

The loader should fail startup for required service config and secrets. Only low-risk values should have defaults, and even those defaults should go through parsing and validation.

## Review Questions

- Which variables are required?
- Which variables may safely default?
- Which variables are secrets?
- Which values are being silently repaired instead of rejected?
- Does the rest of the app still read `process.env` directly?
