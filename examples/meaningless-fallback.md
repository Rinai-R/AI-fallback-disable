# Meaningless Fallback

## Bad AI Code

```ts
const apiKey = process.env.OPENAI_API_KEY?.trim() || "";
const databaseUrl = process.env.DATABASE_URL?.trim() || "localhost";
const jwtSecret = process.env.JWT_SECRET || "secret";
const displayName = user.profile?.displayName || "Unknown";
```

## What It Hides

These fallbacks do not express a real policy. They only let code continue when the input or deployment is wrong.

- `""` is not a usable API key.
- `"localhost"` is not a safe production database URL.
- `"secret"` is not a real signing secret.
- `"Unknown"` is unsupported unless an existing UI policy defines it, and it is meaningless if the name is used for contracts, audit logs, invoices, or identity.

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

For business data, decide whether absence is allowed:

```ts
const displayName = requireProfileDisplayName(user.profile);
```

or keep a presentation-only default only when an existing UI requirement or snapshot/contract test defines that copy:

```ts
const label = user.profile?.displayName ?? "Unnamed user";
```

The difference matters. Proven UI copy can have a display default; identity data should not be invented. Do not add `"Unnamed user"` just because it seems user-friendly.

## Review Questions

- What does the fallback value mean?
- What pre-existing requirement, contract, or test proves that meaning?
- Would the app work correctly with that value?
- Is this default valid in production?
- Should this be optional, required, dev-only, or a typed business absence?
- If the fallback has no meaning, why is it in the code?
