# Meaningless Fallback

## Bad AI Code

```ts
const apiKey = process.env.OPENAI_API_KEY?.trim() || "";
const databaseUrl = process.env.DATABASE_URL?.trim() || "localhost";
const jwtSecret = process.env.JWT_SECRET || "secret";
```

## What It Hides

These fallbacks do not express a real policy. They only let the app start when deployment is wrong.

- `""` is not a usable API key.
- `"localhost"` is not a safe production database URL.
- `"secret"` is not a real signing secret.

## Better Semantics

Required config should fail startup:

```ts
const apiKey = requireTrimmedString(env, "OPENAI_API_KEY");
const databaseUrl = requireUrl(env, "DATABASE_URL");
const jwtSecret = requireSecret(env, "JWT_SECRET");
```

If a value is truly optional, model it as optional instead of faking it:

```ts
const sentryDsn = optionalTrimmedString(env, "SENTRY_DSN");
```

## Review Questions

- What does the fallback value mean?
- Would the app work correctly with that value?
- Is this default valid in production?
- Should this be optional, required, or dev-only?
- If the fallback has no meaning, why is it in the code?
