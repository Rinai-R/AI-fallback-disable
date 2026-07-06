# Environment Config

## Bad AI Code

```ts
const config = {
  port: Number(process.env.PORT) || 3000,
  databaseUrl: process.env.DATABASE_URL?.trim() || "localhost",
  jwtSecret: process.env.JWT_SECRET?.trim() || "secret",
  featureEnabled: Boolean(process.env.FEATURE_ENABLED || false),
};
```

## Why It Hides Failure

- `Number(value) || 3000` treats missing, malformed, `NaN`, and sometimes valid falsy values as the same case.
- `DATABASE_URL || "localhost"` can send production traffic to the wrong place instead of failing startup.
- `JWT_SECRET || "secret"` creates a weak credential boundary.
- Trimming a secret may change an exact-match credential and hides deployment mistakes.
- `Boolean("false")` is true in JavaScript, so boolean config is not actually parsed.

## Better Semantics

```ts
export const config = loadConfig(process.env);
```

The config loader should:

- read environment variables in one module,
- validate at startup,
- fail fast for required config,
- reject secret defaults,
- reject unexpected whitespace around secrets,
- parse booleans from an explicit allowlist such as `true/false`,
- parse numbers with range and unit checks.

## Review Questions

- Which values are low-risk operational defaults, and which are required production config?
- Does this code distinguish missing, blank, malformed, and valid falsy?
- Is any secret silently defaulted, trimmed, logged, or normalized?
- Will a bad deployment fail before serving traffic?
- Do tests cover missing, blank, malformed, valid falsy, and secret whitespace cases?
