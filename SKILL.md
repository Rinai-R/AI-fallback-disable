---
name: ai-fallback-disable
description: Review, implement, or harden AI-assisted software development work with explicit engineering guardrails around defensive code, fallback behavior, configuration loading, secrets, error semantics, agent workflows, verifier checks, and release readiness. Use when AI-generated code adds broad try/catch, silent defaults, optional chaining noise, unsafe environment-variable handling, vague "make it robust" changes, or when turning repeated AI coding lessons into reusable scripts, lint rules, tests, or skills.
---

# AI Fallback Disable

## Purpose

Use this skill to keep AI-assisted development explicit, reviewable, and production-safe. The goal is not to ban defensive programming; the goal is to put defense at system boundaries, keep core logic contract-driven, and prevent AI-generated fallback code from hiding real failures.

This skill is based on three practical observations:

- AI often lacks full system context, so it adds `null` checks, `try/catch`, `trim()`, optional chaining, and fallback defaults to make local code appear safe.
- The dangerous part is not defensive code itself; it is silent recovery from errors that should fail fast, alert, or be handled by a deliberate product policy.
- Repeated lessons should move from prompt-only advice into scripts, lint checks, tests, verifier gates, and reusable skills.

## Default Rule

Do not make code "safe" by hiding failures.

Make failures explicit at boundaries, keep core logic clear, and require every fallback to have a named business or operations policy.

## Workflow

1. Identify system boundaries.
2. Identify core logic contracts.
3. Search for AI defensive-code smells.
4. Classify each fallback, default, optional chain, and catch block.
5. Move parsing and validation into boundary modules.
6. Remove silent fallback from trusted core logic unless explicitly justified.
7. Add tests for missing, invalid, and valid falsy values.
8. Run the verifier checklist.
9. If the same issue appears repeatedly, add lint/script coverage or update this skill.

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

- Keep core logic focused on the main domain path.
- Treat violated invariants as bugs or contract failures, not as cases to smooth over with empty values.
- Avoid repeated `if (!x)`, `?.`, and default objects for data that the boundary layer should guarantee.
- Do not use broad `try/catch` around unrelated core operations.
- Use typed/domain results for expected business absence, such as not found, feature disabled, validation failure, or optional capability unavailable.
- Throw or propagate unexpected system failures with context.

## Fallback Classification

Classify every fallback before approving it.

| Category | Examples | Preferred handling |
| --- | --- | --- |
| Required product behavior | Cached read when live source is down, feature disabled by flag | Make it explicit, observable, tested, and documented |
| Low-risk operational default | `PORT`, `LOG_LEVEL`, timeout, page size | Allow only after type/range validation |
| Expected business absence | User not found, optional profile missing | Return typed/domain result, not ambiguous empty values |
| Broken config or deployment | Missing `DATABASE_URL`, `JWT_SECRET`, API key | Fail fast at startup |
| Programmer error | Impossible state, missing required argument | Throw/assert; fix caller or contract |
| Dependency/system failure | Database down, API timeout, queue failure | Propagate with context; retry/circuit-break only by policy |
| AI-generated convenience | `catch { return [] }`, `"secret"` default, local URL fallback | Remove or replace with explicit failure semantics |

## Unsafe Patterns

Flag these unless a local contract clearly justifies them:

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

## Agent Workflow

Use prompt, script, subagent, verifier, and skill at different layers:

| Situation | Best mechanism |
| --- | --- |
| Human judgment, architecture boundaries, error semantics | Prompt or skill instruction |
| Repeated text patterns or mechanically detectable smells | Script or lint rule |
| Large repo audit requiring multiple perspectives | Subagents |
| Final quality gate after code changes | Verifier checklist |
| Cross-project repeated workflow | Skill |
| One local bug | Direct fix plus focused tests |

### Prompt

Use prompt guidance for rules that need judgment:

- Define which layer is a boundary and which layer is trusted core logic.
- Say which errors must fail fast and which outcomes are recoverable business states.
- Require explicit fallback justification.
- Require tests for failure semantics, not just "does not throw".

### Script Or Lint

Create deterministic checks when a bad pattern is searchable:

```bash
rg "process\\.env"
rg "\\|\\|.*default|\\?\\?.*default"
rg "catch\\s*\\{|catch\\s*\\("
rg "return\\s+(null|undefined|\\[\\]|false)"
rg "\\.trim\\(\\)"
rg "Number\\(process\\.env|parseInt\\(process\\.env|parseFloat\\(process\\.env"
rg "as any|unknown as"
```

Turn repeated findings into named checks such as:

- `lint:env-boundary` for scattered environment reads.
- `lint:secret-defaults` for secret fallback and unsafe trim.
- `lint:defensive-fallbacks` for empty catch and fallback returns.
- `lint:unsafe-config-parse` for loose number, boolean, enum, and URL parsing.

### Subagents

Use subagents when the codebase is large or needs parallel review perspectives:

- Env/config and startup boundaries.
- Fallback, `try/catch`, and optional chaining.
- Core logic contract pollution.
- Tests that validate fallback instead of failure semantics.
- Security, permission, and release risks.

Ask each subagent to return findings with severity, file/line, hidden failure, production impact, suggested fix, and whether the issue can be scripted.

### Verifier

Use a verifier after changes to answer: did this actually remove silent failure, or did it move the same ambiguity somewhere else?

The verifier should not keep rewriting. It should check and reject:

- Scattered `process.env`.
- `|| default` or `?? default` on required config.
- Secret defaults or secret trim.
- Catch blocks that swallow unknown failures.
- Fallbacks without product semantics or observability.
- Tests that only assert "does not throw".

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

## Review Output

When reviewing code, lead with findings:

```text
- Severity: Critical | High | Medium | Low
  Location: file:line
  Problem: what failure is being hidden or conflated
  Impact: what can go wrong in production
  Fix: concrete replacement behavior or code direction
  Automation: whether a lint/script/test should catch this next time
```

Prioritize:

1. Security or credential fallback.
2. Data loss, data corruption, tenant/auth boundary errors.
3. Hidden deployment/configuration failures.
4. Swallowed dependency/system failures.
5. Ambiguous business absence or API semantics.
6. Style-level defensive noise.

## Release Gate

Before shipping AI-generated code, confirm:

- Boundaries are explicit and validated.
- Core logic is not full of speculative fallback.
- Failure modes are distinct: config error, validation error, auth error, dependency failure, not found, and programmer error.
- Secrets and credentials never default, never leak, and are not silently normalized.
- Fallbacks are observable and user/product semantics are clear.
- Rollback, feature flags, migrations, queues, caches, and external contracts are considered for risky changes.
- CI includes type checks, lint, unit tests, focused integration tests, and relevant security scans.

For deeper launch checks, read `references/release-checklist.md`.
