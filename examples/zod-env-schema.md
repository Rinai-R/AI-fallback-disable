# Zod Env Schema

## Better Semantics

If the project already uses Zod, use it as the boundary parser. Do not just add `.trim().default()` everywhere.

```ts
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),

  PORT: z
    .string()
    .optional()
    .transform((value) => {
      if (value === undefined || value.trim() === "") {
        return 3000;
      }

      const parsed = Number(value);
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
      if (value === undefined || value.trim() === "") {
        return 5000;
      }

      const parsed = Number(value);
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
- Low-risk defaults are explicit.
- Secret whitespace is rejected, not silently fixed.
- Business code imports `config` instead of reading `process.env`.

## Review Questions

- Does the schema distinguish secrets from normal strings?
- Are defaults limited to low-risk values?
- Are numeric values parsed and range-checked?
- Are errors raised during startup?
- Does the exported config have the right types?
