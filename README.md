# AI Fallback Disable

> [中文说明](README.zh-CN.md)

`ai-fallback-disable` is a Codex skill for auditing and hardening AI-generated code that looks locally robust but hides real failures through unsafe fallback values, swallowed errors, config defaults, secret fallbacks, ambiguous empty returns, or optional chaining that weakens domain contracts.

Core principle:

> Do not make code "safe" by hiding failures.

Reliable code should validate inputs and configuration at boundaries, keep core logic contract-driven, and ensure every fallback has explicit product or operations semantics.

## When To Use

Use this skill when:

- AI-generated code returns fallback values, empty arrays, empty objects, or `null` to avoid visible failure.
- `try/catch` swallows unknown errors or turns dependency failures into success-like results.
- `process.env.X || default` or `Number(value) || default` conflates missing, invalid, and valid falsy values.
- Secrets such as `JWT_SECRET`, API keys, tokens, or signing keys have defaults or are silently trimmed.
- Required context such as `user?.id`, `tenant?.id`, or `permission?.role` is hidden behind optional chaining.
- A vague "make this robust" request needs to become explicit failure semantics, tests, and verifier checks.

## When Not To Use

This is not a generic code review checklist or a broad release-governance skill.

Do not use it to:

- Review ordinary style issues.
- Mechanically delete every fallback.
- Turn every business-level absence into an exception.
- Default to generating grep scripts or lint rules.
- Replace existing project security, release, or architecture standards.

## Usage

In a Codex environment with skills enabled, invoke it explicitly:

```text
Use $ai-fallback-disable to review this AI-generated config loader for unsafe fallback and unclear failure semantics.
```

Or:

```text
Use $ai-fallback-disable to harden this API handler. Focus on swallowed errors, ambiguous empty returns, and tenant/auth contract violations.
```

## Workflow

The skill guides the agent to:

1. Identify the relevant boundary: config, request input, external API, file, database/cache/queue response, identity/tenant/permission context.
2. State the core contract that should hold after boundary validation.
3. Classify suspicious fallback, default, catch, optional chain, or null/empty return.
4. Decide the correct behavior: fail startup, validation error, typed business result, propagated system error, or explicit degraded mode.
5. Make the smallest safe change without breaking valid business-level absence handling.
6. Add focused failure-semantics tests.

## Repository Layout

```text
.
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── agents/
│   └── openai.yaml
└── references/
    └── release-checklist.md
```

- `SKILL.md`: Core skill instructions.
- `agents/openai.yaml`: Display metadata and default prompt.
- `references/release-checklist.md`: Use only when the change touches production release readiness, secrets, auth, external input, deployment, CI, monitoring, rollback, or related operational concerns.

## Design Notes

This skill intentionally avoids making grep/scripts the default move. Automation is useful only for repeated, low-false-positive, stable, high-risk patterns. For normal review, first understand the boundary, contract, and failure semantics, then decide whether a lint rule, script, or verifier gate is justified.
