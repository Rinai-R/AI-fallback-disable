# Zod Env Schema

## Better Semantics

If the project already uses Zod, use it as the boundary parser. Do not just add `.trim().default()` everywhere.

This example assumes an existing configuration policy explicitly defines `PORT=3000` and `REQUEST_TIMEOUT_MS=5000` when those variables are absent. Without that evidence, do not add these defaults.

```ts
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"]),

  PORT: z
    .string()
    .optional()
    .transform((value) => {
      if (value === undefined) {
        return 3000;
      }

      const normalized = value.trim();
      if (normalized === "") {
        throw new Error("PORT must not be blank");
      }

      const parsed = Number(normalized);
      if (!Number.isInteger(parsed) || parsed <= 0 || parsed > 65535) {
        throw new Error("PORT must be an integer between 1 and 65535");
      }

      return parsed;
    }),

  DATABASE_URL: z
    .string()
    .trim()
    .min(1, "DATABASE_URL is required"),

  JWT_SECRET: z
    .string()
    .min(1, "JWT_SECRET is required")
    .refine((value) => value === value.trim(), {
      message: "JWT_SECRET must not contain leading or trailing whitespace",
    }),

  REQUEST_TIMEOUT_MS: z
    .string()
    .optional()
    .transform((value) => {
      if (value === undefined) {
        return 5000;
      }

      const normalized = value.trim();
      if (normalized === "") {
        throw new Error("REQUEST_TIMEOUT_MS must not be blank");
      }

      const parsed = Number(normalized);
      if (!Number.isInteger(parsed) || parsed <= 0) {
        throw new Error("REQUEST_TIMEOUT_MS must be a positive integer");
      }

      return parsed;
    }),
});

export const config = envSchema.parse(process.env);
```

## Why This Works

- Environment variables are parsed once at startup.
- Required values fail fast.
- The two low-risk defaults are preserved from an explicit existing policy and cover absence only.
- Blank values remain invalid instead of being treated as missing.
- Secret whitespace is rejected, not silently fixed.
- Business code imports `config` instead of reading `process.env`.

## Review Questions

- Does the schema distinguish secrets from normal strings?
- Can every default be traced to existing policy rather than "low risk" alone?
- Are numeric values parsed and range-checked?
- Are errors raised during startup?
- Does the exported config have the right types?
