---
name: ai-fallback-disable
description: Audit or harden AI-generated code that hides failures through unsafe fallback values, swallowed errors, scattered config reads, secret defaults, ambiguous null/empty returns, or optional chaining that weakens domain contracts. Use when reviewing AI-written code, removing silent recovery, centralizing config validation, or adding verifier checks for explicit failure semantics.
---

# AI Fallback Disable

## Purpose

Use this skill to review or harden AI-assisted code that looks robust locally but weakens real failure semantics. The goal is not to ban defensive programming; the goal is to stop accidental silent recovery from hiding configuration errors, authorization gaps, dependency failures, malformed input, or broken domain contracts.

Default rule:

> Do not make code "safe" by hiding failures.

Make failures explicit at boundaries, keep core logic contract-driven, and require every fallback to have a clear business or operations meaning.

## Operating Model

Work in this order:

1. Identify the relevant boundary: config, request input, external API, file, database/cache/queue response, identity/tenant/permission context, or other untrusted source.
2. State the core contract that should hold after that boundary has been validated.
3. Classify each suspicious fallback, default, catch block, optional chain, or null/empty return.
4. Decide the correct behavior: fail startup, return validation error, return typed business absence, propagate system error, or use explicit degraded mode.
5. Make the smallest change that restores explicit behavior without breaking valid business-level absence handling.
6. Add focused tests for the failure semantics changed by the fix.
7. Use `references/release-checklist.md` only when the change touches production release paths, secrets, auth, external input, deployment, CI, or operational readiness.

For similar concrete cases, read only the matching example in `examples/`. Examples are judgment references, not copy-paste templates:

- `examples/env-config.md` for environment variables, defaults, secrets, and startup validation.
- `examples/swallowed-db-error.md` for database or dependency errors hidden as empty results.
- `examples/optional-chaining-tenant-context.md` for auth, tenant, or permission context hidden by optional chaining.
- `examples/typed-business-absence.md` for legitimate not-found or empty-result semantics.
- `examples/dependency-degradation-policy.md` for explicit degraded mode versus accidental fallback.

## Boundary Rules

Validate and normalize only at untrusted boundaries:

- Environment variables and startup config.
- CLI arguments.
- HTTP request body, query, headers, cookies, and sessions.
- Webhook payloads.
- Files, JSON/YAML/TOML config, and user uploads.
- Database, cache, queue, and third-party API responses.
- Feature flags, tenant context, auth context, and permission claims.

After boundary validation, pass typed values into core logic. Avoid letting raw request objects, `process.env`, unknown JSON, or loosely typed dependency responses leak into domain code.

## Core Contract Rules

- Keep core logic focused on domain behavior, not speculative recovery.
- Treat violated invariants as contract failures, not cases to smooth over with default objects.
- Avoid repeated `if (!x)`, `?.`, and empty returns for data the boundary layer should guarantee.
- Do not use broad `try/catch` around unrelated core operations.
- Use typed/domain results for expected business absence: not found, disabled feature, optional capability unavailable, empty-but-valid result, or validation failure.
- Throw or propagate unexpected system failures with enough context for callers and observability.

## Fallback Classification

Classify every fallback before approving or removing it.

| Category | Examples | Preferred handling |
| --- | --- | --- |
| Required product behavior | Cached read when live source is down, feature disabled by flag | Make it explicit, observable, tested, and documented |
| Low-risk operational default | `PORT`, `LOG_LEVEL`, timeout, page size | Allow only after type/range validation |
| Expected business absence | User not found, optional profile missing | Return typed/domain result, not ambiguous empty values |
| Broken config or deployment | Missing `DATABASE_URL`, `JWT_SECRET`, API key | Fail fast at startup |
| Programmer error | Impossible state, missing required argument | Throw/assert; fix caller or contract |
| Dependency/system failure | Database down, API timeout, queue failure | Propagate with context; retry/circuit-break only by explicit policy |
| AI-generated convenience | `catch { return [] }`, `"secret"` default, local URL fallback | Remove or replace with explicit failure semantics |

## Unsafe Patterns

Flag these unless the surrounding contract clearly justifies them:

- `process.env.X || default` for config.
- `process.env.X?.trim() || ""` for required values.
- Default secrets such as `"secret"`, `"changeme"`, `"dev-key"`, `"test-key"`, or local credentials.
- `Number(value) || default`, because missing, invalid, `NaN`, and valid `0` collapse into one path.
- Boolean parsing that treats any non-empty string as `true`.
- `.trim()` on secret, token, signature, private key, exact-match credential, or idempotency key.
- `catch {}` or `catch (e) { return null }`.
- Broad `try/catch` around unrelated operations.
- Optional chaining on required identity, tenant, permission, or domain state.
- Empty strings, empty arrays, or empty objects representing multiple failure causes.
- Fallback from failed remote services to mock, local, stale, or empty data without product/ops policy.

## Environment Config Standards

- Read environment variables only in a dedicated config module.
- Parse and validate config during application startup.
- Export a typed config object; business code should not read `process.env` directly.
- Allow defaults only for low-risk tuning: `PORT`, `LOG_LEVEL`, request timeout, page size, default-off feature flags.
- Forbid defaults for `DATABASE_URL`, API keys, auth/session/JWT secrets, encryption keys, OAuth client secrets, webhook secrets, tenant IDs, signing keys, and production endpoints.
- For normal URLs, enums, and whitespace-insensitive strings: trim, then validate.
- For secrets, tokens, signatures, private keys, and exact-match credentials: do not trim; reject unexpected leading/trailing whitespace.
- For numbers: distinguish missing, blank, invalid, zero, negative, and out-of-range values.
- For booleans: parse explicit allowed strings such as `true/false`, `1/0`, or `yes/no`; reject unknown values.
- For URLs: parse with URL tooling and validate protocol, host, and required path semantics.

## Required Review Output

When reviewing code, lead with findings. For each finding, use this structure:

```text
- Severity: Critical | High | Medium | Low
  Location: file:line
  Pattern: unsafe fallback | swallowed error | unsafe config | secret handling | contract pollution | ambiguous absence
  Hidden failure: what failure is currently disguised or conflated
  Correct behavior: fail startup | validation error | typed business result | propagated system error | explicit degraded mode
  Fix: concrete replacement behavior or code direction
  Test: missing/blank/invalid/falsy/dependency-failure case to add
```

Prioritize:

1. Security or credential fallback.
2. Data loss, data corruption, tenant/auth boundary errors.
3. Hidden deployment/configuration failures.
4. Swallowed dependency/system failures.
5. Ambiguous business absence or API semantics.
6. Style-level defensive noise.

## Do Not

- Do not replace every fallback with `throw`.
- Do not remove valid business-level absence handling.
- Do not normalize secrets, signatures, tokens, exact-match credentials, or idempotency keys.
- Do not turn dependency outages into empty success results.
- Do not add broad `try/catch` as a robustness fix.
- Do not collapse unauthorized, forbidden, not found, validation error, and empty result into the same return value.
- Do not add default production endpoints, local service URLs, mock credentials, or weak secrets.
- Do not change public API semantics without identifying the caller impact.
- Do not treat "tests pass" as proof that failure semantics are correct.

## Automation Guidance

Do not start by writing scripts or treating text search as the deliverable. First understand the boundary, contract, and intended failure semantics.

Only propose lint rules, scripts, or verifier checks when a problem is:

- repeated across multiple files or projects,
- mechanically detectable with low false positives,
- risky enough to justify a gate, and
- stable enough that future agents should not rediscover it by judgment alone.

Examples of rules that may become automation after repeated findings:

- environment variables are read outside the config module,
- secrets or production endpoints have defaults,
- required config uses fallback defaults,
- catch blocks swallow unknown errors,
- optional chaining hides required identity or tenant context.

Keep automation as a follow-up hardening path, not the default answer to every review.

## Testing Requirements

Add focused tests for:

- Required config missing.
- Config blank.
- Config malformed.
- Valid falsy values such as `false` or `0`.
- Secret missing.
- Secret with leading/trailing whitespace.
- URL, enum, boolean, and number parsing failure.
- Unauthorized, forbidden, cross-tenant, and expired credential paths.
- External dependency failure behavior.
- Unknown exceptions not being swallowed.

Good test names:

```text
throws when required env is missing
throws when PORT is not an integer
does not replace false with default
does not silently trim API_TOKEN
does not read process.env outside config loader
does not swallow unexpected calculation errors
```

## Verifier Checklist

Use this checklist after edits:

- Does the boundary now parse and validate untrusted input before core logic receives it?
- Are missing, blank, invalid, and valid falsy values distinct where they matter?
- Are secrets free of defaults and silent normalization?
- Does expected business absence use a typed/domain result instead of ambiguous empty values?
- Are unexpected dependency or system failures propagated with context?
- Does every remaining fallback have explicit product or operations semantics?
- Can callers, logs, metrics, or tests observe degraded mode when it happens?
- Do tests assert the intended failure behavior, not merely "does not throw"?

## Release Gate

Use the release checklist only when the reviewed change affects production readiness, including secrets, auth, external input, deployment, CI, monitoring, rollback, migrations, queues, caches, or external contracts.

For those cases, read `references/release-checklist.md` and apply only the sections relevant to the change.
