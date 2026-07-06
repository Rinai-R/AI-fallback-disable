---
name: ai-fallback-disable
description: Audit, write, or refactor AI-generated code that hides failures through meaningless fallback, wrong fallback, swallowed errors, ambiguous null/empty results, unsafe defaults, secret/config fallback, or optional chaining that weakens domain contracts. Use when reviewing fallback semantics in business logic, boundary input handling, dependency errors, config loading, auth/tenant/payment flows, or prompts that should prevent AI from inventing values just to keep code running.
---

# AI Fallback Disable

## Purpose

Use this skill when AI is writing or reviewing code that adds fallback behavior.

The core problem is not that AI writes defensive code. The problem is that it often invents a local fallback without understanding the boundary, the domain contract, or the failure semantics.

Hard rule:

> Meaningless fallback should not exist in code.

A fallback is meaningful only when it has an explicit product, domain, or operational policy. A fallback that merely keeps the program running, returns a normal-looking value, or hides the real failure is not robustness.

## Fallback Taxonomy

Classify fallback by the semantics it changes, not by syntax.

| Type | Meaning | Typical Fix |
| --- | --- | --- |
| Meaningless fallback | Fills a value with no real policy, only to avoid `undefined`, `null`, or an exception | Remove it; model required or optional explicitly |
| Wrong fallback | Produces a value with the wrong business meaning | Replace with validation, explicit error, or correct domain state |
| Swallowed-error fallback | Converts system failure into a normal result | Propagate the error, return typed failure, or make degradation visible |
| Semantic-confusion fallback | Uses one value for multiple states such as not found, forbidden, empty, and dependency failure | Split the states with typed results or explicit errors |
| Contract-breaking fallback | Weakens a core invariant after the boundary should already be validated | Move validation to the boundary and keep core logic strict |
| Unsafe boundary fallback | Defaults config, identity, tenant, auth, payment, storage, or secret values | Fail fast or require an explicit environment-gated policy |
| Valid fallback | Expresses a real product or operational rule and does not hide errors | Keep it, name it, validate it, and test it |

Good fallback is policy. Bad fallback is pretending an error is fixed by inventing a value.

## Suspect Patterns

Treat these as suspect until their semantics are proven:

- `process.env.API_KEY?.trim() || ""`
- `process.env.DATABASE_URL || "localhost"`
- `process.env.JWT_SECRET || "secret"`
- `Number(process.env.PORT) || 3000` without distinguishing missing, invalid, and out-of-range values
- `process.env.X ?? default` on required config or production boundary values
- `user.role || "user"` when missing role means corrupt identity data
- `product.price || 0` when missing price is not a free product
- `tenantId || "default"` in multi-tenant code
- `permissionError ? false : value` when dependency failure should not look like denial
- `catch { return null }`, `catch { return [] }`, `catch { return {} }`, or `catch { return false }`
- `optional?.chain ?? default` on values that should be guaranteed by a validated contract
- empty string, zero, empty list, empty object, local URL, mock token, weak secret, or fake identity used to keep execution alive

Do not keep meaningless fallback and document around it. If the value is required, fail at the boundary. If it is optional, model absence explicitly. If it has a real default, name the policy and validate it.

## Workflow

1. Identify the boundary: request, config, file, queue message, database row, dependency response, cache, auth context, tenant context, or UI input.
2. Identify the core contract after that boundary: which fields must exist, which states are allowed, and which failures must remain visible.
3. Classify each fallback with the taxonomy above before editing code.
4. Ask what exact failure the fallback hides: missing input, invalid input, not found, forbidden, dependency failure, corrupt data, deploy error, or optional absence.
5. Decide the correct behavior: fail fast, validation error, typed business absence, propagated system error, explicit degraded mode, or validated default.
6. Move validation and normalization to the boundary. Keep core logic on typed, already-validated values.
7. Remove meaningless and wrong fallback. Preserve valid fallback only when it has a named policy.
8. Add tests for the states that were previously collapsed.

When a similar case appears, read only the relevant example:

- `examples/meaningless-fallback.md` for fallback values that should simply be removed.
- `examples/wrong-fallback.md` for fallback that creates false business facts.
- `examples/swallowed-error-fallback.md` for `catch` blocks that turn system errors into normal results.
- `examples/semantic-confusion.md` for `null`, `[]`, `{}`, or `false` representing too many states.
- `examples/valid-fallback.md` for defaults and degradation that are legitimate policy.
- `examples/bad-env-loader.md` for config-specific `trim`, defaults, and loose parsing.
- `examples/secret-trim.md` for secrets, tokens, private keys, and exact-match credentials.
- `examples/default-policy.md` for deciding which config may have defaults.
- `examples/zod-env-schema.md` for schema-based startup validation.

Examples are judgment references, not templates to paste blindly.

## Classification Questions

Ask these before accepting any fallback:

- Is this value a business rule, product default, operational policy, or just an invented placeholder?
- Does the fallback hide a deployment error, invalid input, corrupt data, dependency outage, permission problem, or security boundary failure?
- Would the fallback result be treated later as a real fact: identity, role, price, balance, inventory, tenant, region, permission, or state?
- Are `missing`, `blank`, `invalid`, `not found`, `empty`, `forbidden`, and `dependency failed` distinguishable?
- Does the caller need to know that degradation happened?
- Is the fallback observable through logs, metrics, status, or typed result when it handles a real outage?
- Would continuing be safer than failing?

## Business Logic Rules

- Do not default identity, role, tenant, account, region, resource ID, permission, price, balance, inventory, billing, or state-machine values unless a domain policy explicitly says so.
- Do not turn dependency failure into business absence. Database down is not user not found. Payment service timeout is not zero balance. Permission service failure is not ordinary forbidden.
- Do not use `null`, `false`, `[]`, or `{}` for every unhappy path. Split business absence from system failure.
- Do not keep optional chaining inside core logic to compensate for a boundary that should have validated the object.
- Do not replace every fallback with `throw`. Business absence and product defaults are valid when they are explicit and tested.
- Prefer typed results for expected business states: `found/not_found`, `allowed/denied`, `empty`, `disabled`, `degraded`, `dependency_error`.

## Config And Secret Rules

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

## Trim Policy For Config

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

## Default Policy For Config And Inputs

Use defaults only when all are true:

- the default is a named product, domain, or operational policy,
- it covers absence, not invalidity or system failure,
- it is low-risk in the target environment,
- it does not affect identity, permission, money, tenant, storage, or security boundaries,
- it is parsed and validated when it comes from outside the process,
- tests cover missing and invalid cases separately.

Do not default:

- secrets,
- API keys,
- database URLs,
- production service endpoints,
- tenant IDs,
- user IDs,
- roles and permissions,
- prices, balances, inventory, and billing facts,
- state-machine statuses,
- payment, storage, and data-routing targets,
- signing keys,
- webhook secrets,
- encryption material,
- values that change security, identity, billing, or data storage boundaries.

Local development defaults are allowed only when explicitly scoped to development. They must not be reachable in production by accidentally omitting an environment variable.

## Parse And Boundary Rules

- Do not use `X || default` for config parsing.
- Distinguish `undefined`, blank string, malformed value, out-of-range value, and valid falsy value.
- Parse numbers explicitly; validate integer/float, positive/negative, range, and unit.
- Parse booleans from an allowlist; do not rely on JavaScript truthiness.
- Parse URLs with URL tooling; validate protocol and host when relevant.
- Parse enums with an allowlist.
- Return a typed config object so the rest of the app does not carry `string | undefined`.
- For user input, distinguish omitted optional fields from invalid supplied fields.
- For dependency calls, distinguish cache miss from cache failure, not found from database failure, and disabled feature from failed integration.

## Required Review Output

When reviewing fallback behavior, return findings in this shape:

```text
- Severity: Critical | High | Medium | Low
  Location: file:line
  Pattern: meaningless fallback | wrong fallback | swallowed error | semantic confusion | contract-breaking fallback | unsafe config fallback | unsafe trim | loose parse
  Current behavior: what the code currently does
  Hidden failure: what failure or state is being disguised
  Correct behavior: fail fast | validation error | typed business result | propagated system error | explicit degraded mode | validated default
  Fix: concrete code direction
  Test: missing/blank/invalid/not-found/forbidden/dependency-failure/falsy/range/secret-whitespace case to add
```

Prioritize:

1. Auth, tenant, permission, payment, billing, storage, data-routing, and secret fallback.
2. System errors hidden behind `null`, `false`, `[]`, `{}`, zero, or success responses.
3. Wrong business facts such as default role, price, balance, inventory, region, or state.
4. Required config hidden behind empty string, localhost, mock values, weak defaults, or loose parsing.
5. Valid fallback that lacks tests, observability, or a named policy.

## Do Not

- Do not keep fallback just because it makes startup pass.
- Do not replace required config with empty strings.
- Do not invent business facts: role, tenant, price, balance, inventory, or state.
- Do not collapse not found, forbidden, empty, disabled, invalid input, and dependency failure into one value.
- Do not return normal empty data from an unexpected system failure unless it is an explicit degraded mode.
- Do not default secrets to `"secret"`, `"changeme"`, `"dev-key"`, or test credentials.
- Do not silently trim secrets.
- Do not use local URLs as production fallback.
- Do not parse booleans with truthiness.
- Do not use `Number(value) || default` as validation.
- Do not let core business code keep validating raw boundary data.
- Do not add a script or grep check as the main fix; first fix the contract and failure semantics.

## Prompt Template

When asking an AI agent to review fallback behavior, use constraints like:

```text
Review this change for unsafe fallback behavior.

Requirements:
- classify each fallback as meaningful, meaningless, wrong, swallowed-error, semantic-confusion, contract-breaking, unsafe boundary, or valid
- identify what failure each fallback hides
- do not treat all fallback as bad; keep policy-backed defaults and explicit degradation
- do not allow fallback for identity, tenant, permission, money, payment, storage, secret, or production routing values without a named policy
- distinguish missing, invalid, not found, forbidden, empty, disabled, and dependency failure
- recommend typed results, boundary validation, explicit errors, or observable degradation
- include tests for the states that were previously collapsed
```

## Verifier Checklist

Before accepting the change:

- Is each fallback classified by semantics, not syntax?
- Are meaningless fallback values removed instead of documented?
- Are wrong business facts removed: fake role, tenant, price, balance, inventory, state, or endpoint?
- Are system failures still visible to callers, logs, metrics, or typed results?
- Are not found, forbidden, empty, disabled, invalid input, and dependency failure distinguishable?
- Is boundary validation done before core logic?
- Are allowed defaults explicit, low-risk, parsed, and tested?
- Are secrets free of defaults and silent trim?
- Are booleans, numbers, enums, and URLs explicitly parsed?
- Does business code receive typed inputs instead of raw boundary data?
- Do tests cover missing, blank, invalid, valid falsy, not found, forbidden, dependency failure, range, and secret-whitespace cases?
