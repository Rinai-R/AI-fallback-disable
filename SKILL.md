---
name: ai-fallback-disable
description: Audit, write, or refactor AI-generated environment/config loading code that uses meaningless fallback values, unsafe defaults, scattered process.env reads, secret defaults, silent trim, or loose parsing. Use when reviewing or building config modules, startup validation, env schemas, secret handling, or prompts that should prevent process.env.X || default and similar silent configuration recovery.
---

# AI Fallback Disable

## Purpose

Use this skill when AI is writing or reviewing environment/config loading code.

The core problem is not that AI writes defensive code. The problem is that it often adds fallback values without knowing whether the value is optional, required, sensitive, low-risk, or production-critical.

Hard rule:

> Meaningless fallback should not exist in code.

A fallback is meaningful only when it has an explicit product, operational, or low-risk configuration policy. A fallback that merely keeps the program running while hiding a bad deployment is not robustness.

## What Counts As Meaningless Fallback

Treat these as suspect until proven otherwise:

- `process.env.API_KEY?.trim() || ""`
- `process.env.DATABASE_URL || "localhost"`
- `process.env.JWT_SECRET || "secret"`
- `Number(process.env.PORT) || 3000` without distinguishing missing, invalid, and out-of-range values
- `process.env.X ?? default` on required config
- `catch { return defaultConfig }` around config loading
- empty string, local URL, mock token, weak secret, or test credential used to keep startup alive

Remove meaningless fallback instead of documenting it. If the value is required, fail startup. If it is optional, model it as optional. If it has a real default, validate and name that policy.

## Workflow

1. Find where configuration is read.
2. Move `process.env` reads into one config module if they are scattered.
3. Classify each variable before writing parsing code.
4. Decide whether the variable is required, optional, low-risk defaultable, dev-only defaultable, or forbidden to default.
5. Parse and validate during startup.
6. Export a typed config object.
7. Ensure business code uses config, not `process.env`.
8. Add tests for missing, blank, invalid, valid falsy, range, and secret-whitespace cases.

When a similar case appears, read only the relevant example:

- `examples/bad-env-loader.md` for mixed `trim`, defaults, and loose parsing.
- `examples/meaningless-fallback.md` for fallback values that should simply be removed.
- `examples/secret-trim.md` for secrets, tokens, private keys, and exact-match credentials.
- `examples/default-policy.md` for deciding which config may have defaults.
- `examples/zod-env-schema.md` for schema-based startup validation.

Examples are judgment references, not templates to paste blindly.

## Variable Classification

Classify config before implementation:

| Class | Examples | Rule |
| --- | --- | --- |
| Low-risk defaultable | `PORT`, `LOG_LEVEL`, `REQUEST_TIMEOUT_MS`, page size | Default only after explicit parse and range validation |
| Required service config | `DATABASE_URL`, `REDIS_URL`, production API base URL | No default; missing/blank/invalid fails startup |
| Secret or credential | `JWT_SECRET`, `SESSION_SECRET`, `API_KEY`, private key, webhook secret | No default; do not silently trim; reject unexpected whitespace |
| Enum | `NODE_ENV`, region, log format | Trim if safe, then validate against an allowlist |
| Boolean | feature flags, debug toggles | Parse explicit values such as `true/false` or `1/0`; reject unknown strings |
| Dev-only convenience | local URLs, test credentials | Keep outside production config; make the environment condition explicit |
| Optional config | optional integration ID, optional feature endpoint | Represent as optional; do not fake a value |

## Trim Policy

`trim()` is not good or bad by itself. It depends on the variable type.

Usually safe to trim before validation:

- URLs.
- enums.
- hostnames.
- numeric strings.
- whitespace-insensitive normal strings.

Do not silently trim:

- passwords,
- tokens,
- HMAC secrets,
- JWT/session secrets,
- private keys,
- certificates,
- Base64 material,
- signatures,
- idempotency keys,
- exact-match credentials.

If a secret must not contain leading or trailing whitespace, validate that condition and fail startup. Do not repair it silently.

## Default Policy

Use defaults only when all are true:

- the default is low-risk,
- it is valid in the target environment,
- it does not hide deployment failure,
- it is parsed and validated,
- tests cover missing and invalid cases.

Do not default:

- secrets,
- API keys,
- database URLs,
- production service endpoints,
- tenant IDs,
- signing keys,
- webhook secrets,
- encryption material,
- values that change security, identity, billing, or data storage boundaries.

Local development defaults are allowed only when explicitly scoped to development. They must not be reachable in production by accidentally omitting an environment variable.

## Parse Rules

- Do not use `X || default` for config parsing.
- Distinguish `undefined`, blank string, malformed value, out-of-range value, and valid falsy value.
- Parse numbers explicitly; validate integer/float, positive/negative, range, and unit.
- Parse booleans from an allowlist; do not rely on JavaScript truthiness.
- Parse URLs with URL tooling; validate protocol and host when relevant.
- Parse enums with an allowlist.
- Return a typed config object so the rest of the app does not carry `string | undefined`.

## Required Review Output

When reviewing config code, return findings in this shape:

```text
- Severity: Critical | High | Medium | Low
  Variable: ENV_NAME
  Pattern: meaningless fallback | forbidden default | unsafe trim | loose parse | scattered env read
  Current behavior: what the code currently does
  Why it is wrong: what deployment/configuration error is hidden
  Correct behavior: fail startup | explicit optional | validated default | dev-only default
  Fix: concrete code direction
  Test: missing/blank/invalid/falsy/range/secret-whitespace case to add
```

Prioritize:

1. Secret defaults or silent secret normalization.
2. Database URLs, production endpoints, tenant/security boundaries with defaults.
3. Required config hidden behind empty string, localhost, mock values, or weak defaults.
4. Loose number/boolean parsing.
5. Scattered `process.env` reads outside the config module.

## Do Not

- Do not keep fallback just because it makes startup pass.
- Do not replace required config with empty strings.
- Do not default secrets to `"secret"`, `"changeme"`, `"dev-key"`, or test credentials.
- Do not silently trim secrets.
- Do not use local URLs as production fallback.
- Do not parse booleans with truthiness.
- Do not use `Number(value) || default` as validation.
- Do not let business code read `process.env` directly.
- Do not add a script or grep check as the main fix; first fix the config contract.

## Prompt Template

When asking an AI agent to write config loading code, use constraints like:

```text
Write a TypeScript config module.

Requirements:
- all environment variables are read only in this module
- config is parsed and validated during startup
- required config fails startup when missing, blank, or invalid
- PORT and REQUEST_TIMEOUT_MS may have validated low-risk defaults
- DATABASE_URL, JWT_SECRET, API_KEY, and SESSION_SECRET must not have defaults
- normal URLs and enums may be trimmed before validation
- secrets must not be automatically trimmed; leading/trailing whitespace is an error
- do not use process.env.X || default
- numeric config must be explicitly parsed and range-checked
- export a typed config object
- business code must not read process.env directly
```

## Verifier Checklist

Before accepting the change:

- Are all `process.env` reads centralized?
- Is each variable classified?
- Are meaningless fallback values removed?
- Are required variables fail-fast?
- Are allowed defaults low-risk, explicit, parsed, and tested?
- Are secrets free of defaults and silent trim?
- Are booleans, numbers, enums, and URLs explicitly parsed?
- Does business code receive a typed config object?
- Do tests cover missing, blank, invalid, valid falsy, range, and secret-whitespace cases?
