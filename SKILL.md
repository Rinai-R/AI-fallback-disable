---
name: ai-fallback-disable
description: Remove or audit AI-generated fallback code with a deletion-first policy. Use when existing code hides failures through meaningless defaults, wrong fallback, swallowed errors, ambiguous null/empty results, unsafe config or secret defaults, speculative compatibility paths, retries, degraded modes, or optional chaining that weakens domain contracts; also use to stop AI from adding defensive fallback without evidence. Preserve or add fallback only when an explicit product, domain, interface, or operational policy can be cited.
---

# AI Fallback Disable

## Purpose

Use this skill to reduce unsupported fallback behavior, not to make code look more defensive.

AI often sees incomplete context and protects the local function with defaults, guards, `try/catch`, optional chaining, retry, compatibility branches, or alternate data sources. That local self-protection can hide the failure that the system needs to expose.

Hard rules, in order:

1. Remove unsupported fallback before considering any new mechanism.
2. Restore the existing contract with the smallest change. If the contract already guarantees a value, use it directly. If an unexpected error already propagates correctly, let it propagate.
3. Preserve an existing fallback only when repository evidence or the user identifies the policy it implements.
4. Add no fallback during cleanup unless the user explicitly asks for it or an existing cited policy is currently unimplemented.

> The default fallback-addition budget is zero.

A fallback is meaningful only when it implements an explicit product, domain, interface, or operational policy. A fallback that merely prevents an exception, returns a normal-looking value, preserves speculative compatibility, or keeps execution moving is unsupported.

## Fallback Taxonomy

Classify fallback by the semantics it changes, not by syntax.

| Type | Meaning | Typical Fix |
| --- | --- | --- |
| Meaningless fallback | Fills a value with no real policy, only to avoid `undefined`, `null`, or an exception | Remove it; model required or optional explicitly |
| Unsupported fallback | Looks plausible but has no user requirement, contract, test, ADR, runbook, or caller behavior proving the policy | Remove it when the contract is clear; otherwise mark `Needs policy` without inventing a replacement |
| Wrong fallback | Produces a value with the wrong business meaning | Replace with validation, explicit error, or correct domain state |
| Swallowed-error fallback | Converts system failure into a normal result | Propagate the error, return typed failure, or make degradation visible |
| Semantic-confusion fallback | Uses one value for multiple states such as not found, forbidden, empty, and dependency failure | Split the states with typed results or explicit errors |
| Contract-breaking fallback | Weakens a core invariant after the boundary should already be validated | Move validation to the boundary and keep core logic strict |
| Unsafe boundary fallback | Defaults config, identity, tenant, auth, payment, storage, or secret values | Fail fast or require an explicit environment-gated policy |
| Valid fallback | Implements a cited existing policy and does not hide invalid input or an unrelated system failure | Keep it; do not redesign or duplicate it during cleanup |

Good fallback is evidenced policy. Naming a constant, adding a comment, or calling a branch "degraded mode" does not create policy.

## Removal-First Operating Contract

Before editing, inventory the fallback surface in scope:

- value substitution: `||`, `??`, ternary defaults, schema `.default()`, `getOrDefault`, placeholder values;
- exception conversion: `catch` or `.catch()` returning normal data, boolean, zero, `null`, or success;
- contract weakening: redundant guards, optional chaining, or local validation after a boundary already guarantees the value;
- alternate execution: retry, stale cache, alternate provider/source, legacy field, mock data, compatibility path, or silent feature disablement;
- degraded behavior: reduced data or functionality returned after a failure.

For an edit task:

- Start with a fallback-addition budget of `0`.
- Prefer deletion-only patches. Do not replace one fallback with another spelling or move it into a helper.
- Do not add guards, assertions, validators, wrappers, `try/catch`, retry, logging, metrics, feature flags, compatibility branches, or alternate providers merely because they seem safer.
- Reuse an existing boundary parser or domain result when one already exists. Do not build a parallel defensive layer.
- Add boundary validation only when the boundary is in scope, the input is genuinely unvalidated, and the removed fallback would otherwise make the contract ambiguous. Add it once at the owning boundary.
- Do not create a typed result unless the domain already distinguishes those states and callers need to handle them. A new union is not a substitute for deleting a swallowed error.
- If policy is unclear and changing it would alter a public contract, return `Needs policy` with the exact missing decision. Do not guess and do not add a temporary fallback.

The patch should normally reduce the fallback surface. A non-negative fallback delta requires explicit justification.

## Evidence Gate

Accept evidence that existed before the cleanup:

- an explicit user or product requirement;
- a type, schema, API contract, protocol, or compatibility matrix;
- a contract test that asserts the missing or failure behavior;
- an ADR, runbook, SLO, or configuration policy;
- existing caller behavior that distinguishes a default or `degraded` result.

Do not accept these as evidence:

- "this is safer," "common practice," or "better UX";
- the AI's own assumption about what users probably want;
- a new constant, helper name, comment, log, metric, or test written in the same patch;
- the fact that removing fallback can expose an error;
- low risk by itself. Low risk is necessary for some defaults, not sufficient authorization.

To retain or add a fallback, identify all of the following:

1. Evidence: where the policy is defined.
2. Trigger: the exact state it covers, such as absence, timeout, or a documented legacy version.
3. Semantics: why the substitute remains a truthful result for that state.
4. Boundary: why this layer owns recovery.
5. Visibility: how callers can distinguish degradation when a system failure is involved.
6. Tests: which existing or required contract cases cover it.

If these cannot be answered, do not add fallback. Do not keep an unsafe fallback merely because evidence is missing; inspect types, callers, tests, and boundary validation to determine whether direct deletion is correct.

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

Do not keep meaningless fallback and document around it. If the value is required, fail at the boundary. If it is optional, model absence explicitly. If it has a real default, cite the existing policy and validate it.

## Workflow

1. Identify the boundary and read the nearby types, schemas, callers, tests, and error handling before editing.
2. Inventory the existing fallback surface and record a baseline count. Do not propose new fallback yet.
3. Identify the core contract after the boundary: required fields, allowed absence, expected business states, and failures that must remain visible.
4. Classify every existing fallback as `Remove`, `Keep`, or `Needs policy`, and cite evidence for every `Keep`.
5. Remove meaningless, unsupported, wrong, swallowed-error, semantic-confusion, and contract-breaking fallback.
6. Use the smallest correct replacement: direct access to a guaranteed value, natural error propagation, an existing boundary parser, or an existing domain result.
7. Preserve valid fallback unchanged when possible. Do not generalize it to neighboring values or failure states.
8. Add fallback only if it passes the Evidence Gate. Otherwise keep the addition count at zero.
9. Add or update focused tests for changed semantics. Do not write a new test to retroactively justify an invented fallback.
10. Inspect the diff and report the fallback delta: removed, retained, added, and net change.

When a similar case appears, read only the relevant example:

- `examples/meaningless-fallback.md` for fallback values that should simply be removed.
- `examples/removal-first-refactor.md` for deleting fallback without replacing it with guards, wrappers, or another fallback.
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
- What pre-existing evidence proves this layer should recover instead of expose the failure?
- Can the fallback be deleted with no replacement because the boundary or type already guarantees the value?
- Is the proposed fix actually a new fallback, compatibility path, retry, or defensive wrapper under another name?

## Business Logic Rules

- Do not default identity, role, tenant, account, region, resource ID, permission, price, balance, inventory, billing, or state-machine values unless a cited domain policy explicitly says so.
- Do not turn dependency failure into business absence. Database down is not user not found. Payment service timeout is not zero balance. Permission service failure is not ordinary forbidden.
- Do not use `null`, `false`, `[]`, or `{}` for every unhappy path. Split business absence from system failure.
- Do not keep optional chaining inside core logic to compensate for a boundary that should have validated the object.
- Do not replace every fallback with `throw`. Business absence and product defaults are valid when they are explicit and tested.
- Use typed results for expected business states only when those distinctions already belong to the domain contract: `found/not_found`, `allowed/denied`, `empty`, or `disabled`. Add `degraded` or `dependency_error` only when an existing interface requires callers to receive those states.

## Config And Secret Rules

Classify config before implementation:

| Class | Examples | Rule |
| --- | --- | --- |
| Potentially defaultable | `PORT`, `LOG_LEVEL`, `REQUEST_TIMEOUT_MS`, page size | Keep or add a default only with cited policy evidence, then parse and validate explicitly |
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

- the policy evidence can be cited from the user request or repository,
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

Do not infer a default merely because a variable is commonly defaulted elsewhere. During cleanup, retain a proven low-risk default; do not introduce neighboring defaults for consistency.

## Parse And Boundary Rules

- Do not use `X || default` for config parsing.
- Distinguish `undefined`, blank string, malformed value, out-of-range value, and valid falsy value.
- Parse numbers explicitly; validate integer/float, positive/negative, range, and unit.
- Parse booleans from an allowlist; do not rely on JavaScript truthiness.
- Parse URLs with URL tooling; validate protocol and host when relevant.
- Parse enums with an allowlist.
- Return a typed config object so the rest of the app does not carry `string | undefined`.
- Prefer the project's existing config boundary. Do not add a second schema, wrapper, or fallback loader when one already owns parsing.
- For user input, distinguish omitted optional fields from invalid supplied fields.
- For dependency calls, distinguish cache miss from cache failure, not found from database failure, and disabled feature from failed integration.

## Required Review Output

When reviewing fallback behavior, return findings in this shape:

```text
- Severity: Critical | High | Medium | Low
  Location: file:line
  Pattern: meaningless fallback | wrong fallback | swallowed error | semantic confusion | contract-breaking fallback | unsafe config fallback | unsafe trim | loose parse
  Disposition: Remove | Keep | Needs policy
  Policy evidence: user requirement | file:line | test:line | none
  Current behavior: what the code currently does
  Hidden failure: what failure or state is being disguised
  Correct behavior: direct contract use | fail fast | validation error | existing typed business result | propagated system error | evidenced degraded mode | evidenced validated default
  Minimal fix: the smallest code direction, preferring deletion with no replacement
  Test: missing/blank/invalid/not-found/forbidden/dependency-failure/falsy/range/secret-whitespace case to add
```

For a code-changing task, finish with:

```text
Fallback delta: removed N, retained N, added N, net +/-N
Added fallback evidence: none | exact policy source for each addition
```

`added N` should normally be `0`. Moving fallback into a helper, catch, retry, validator wrapper, alternate source, compatibility branch, or degraded result still counts as an addition.

Prioritize:

1. Auth, tenant, permission, payment, billing, storage, data-routing, and secret fallback.
2. System errors hidden behind `null`, `false`, `[]`, `{}`, zero, or success responses.
3. Wrong business facts such as default role, price, balance, inventory, region, or state.
4. Required config hidden behind empty string, localhost, mock values, weak defaults, or loose parsing.
5. Existing fallback that may be valid but lacks citable policy evidence.

## Do Not

- Do not keep fallback just because it makes startup pass.
- Do not replace required config with empty strings.
- Do not invent business facts: role, tenant, price, balance, inventory, or state.
- Do not collapse not found, forbidden, empty, disabled, invalid input, and dependency failure into one value.
- Do not return normal empty data from an unexpected system failure unless an existing cited contract defines an explicit degraded mode.
- Do not default secrets to `"secret"`, `"changeme"`, `"dev-key"`, or test credentials.
- Do not silently trim secrets.
- Do not use local URLs as production fallback.
- Do not parse booleans with truthiness.
- Do not use `Number(value) || default` as validation.
- Do not let core business code keep validating raw boundary data.
- Do not add a script or grep check as the main fix; first fix the contract and failure semantics.
- Do not add fallback for symmetry, consistency, future compatibility, defensive completeness, or hypothetical outages.
- Do not replace a removed fallback with a guard plus `throw` when an existing type or boundary already guarantees the value.
- Do not catch and rethrow, wrap, log, or retry an error when natural propagation already preserves the required semantics.
- Do not create a new helper merely to hide the same default or empty result.
- Do not treat a new comment, constant, metric, or test as proof that fallback is product policy.

## Prompt Template

When asking an AI agent to review fallback behavior, use constraints like:

```text
Review this change for unsafe fallback behavior.

Requirements:
- work removal-first: inventory existing fallback before editing and keep the new-fallback budget at zero by default
- classify each fallback as meaningful, meaningless, wrong, swallowed-error, semantic-confusion, contract-breaking, unsafe boundary, or valid
- identify what failure each fallback hides
- remove unsupported fallback with the smallest change; if the existing contract guarantees the value, use it directly without a new guard, wrapper, or throw
- do not treat all fallback as bad; preserve existing fallback only when you can cite pre-existing product, domain, interface, test, or operational policy
- do not invent or add defaults, catch-to-value behavior, retries, alternate providers or sources, compatibility branches, stale-cache paths, or degraded modes
- do not allow fallback for identity, tenant, permission, money, payment, storage, secret, or production routing values without cited policy evidence
- distinguish missing, invalid, not found, forbidden, empty, disabled, and dependency failure
- use typed results or boundary validation only when the existing contract requires them; do not build a parallel defensive layer
- when policy is genuinely unclear and contract-changing, mark Needs policy instead of guessing
- include tests for the states that were previously collapsed
- report fallback delta as removed, retained, added, and net change; justify every addition with an exact evidence source
```

## Verifier Checklist

Before accepting the change:

- Is each fallback classified by semantics, not syntax?
- Was the existing fallback surface inventoried before adding code?
- Are meaningless fallback values removed instead of documented?
- Did simple deletion remain simple, without a replacement guard, catch/rethrow, helper, wrapper, retry, or alternate source?
- Are wrong business facts removed: fake role, tenant, price, balance, inventory, state, or endpoint?
- Are system failures still visible through the existing caller, error, logging, metrics, or result contract?
- Are not found, forbidden, empty, disabled, invalid input, and dependency failure distinguishable?
- When validation is required, is it owned once by the correct boundary rather than duplicated in core logic?
- Are allowed defaults explicit, low-risk, parsed, and tested?
- Does every retained fallback cite policy evidence that predates this patch?
- Is the added-fallback count zero? If not, does every addition pass the Evidence Gate and implement an existing policy?
- Did the patch avoid migrating fallback to another layer or spelling?
- Are secrets free of defaults and silent trim?
- Are booleans, numbers, enums, and URLs explicitly parsed?
- Does business code receive typed inputs instead of raw boundary data?
- Do tests cover missing, blank, invalid, valid falsy, not found, forbidden, dependency failure, range, and secret-whitespace cases?
- Is the fallback delta negative, or explicitly justified when it is zero or positive?
