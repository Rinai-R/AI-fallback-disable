# AI Fallback Disable

> [中文说明](README.zh-CN.md)

Codex skill for reviewing AI-generated fallback code.

The narrow target is code that looks defensive but hides the thing that should have failed: bad config, missing secrets, swallowed dependency errors, broken auth/tenant context, or ambiguous empty returns.

## What It Checks

- Unsafe fallback values.
- Swallowed errors.
- Scattered config reads.
- Secret defaults or secret normalization.
- `null`, empty array, empty object, or `Optional.empty()` used for multiple meanings.
- Optional chaining that hides a required domain invariant.

The main question is always:

> Is this a real business outcome, or did the code disguise a failure?

## What It Is Not

This is not a grep-based scanner. It also is not a general style review skill.

Text search can help find smoke, but the skill is about judgment: boundary, contract, failure type, caller expectation, and observability.

## Usage

```text
Use $ai-fallback-disable to review this config loader. Focus on unsafe defaults, secrets, and failure semantics.
```

```text
Use $ai-fallback-disable to review this handler. Focus on swallowed errors and ambiguous empty returns.
```

## Examples

The `examples/` folder contains short review cases:

- `env-config.md`
- `swallowed-db-error.md`
- `optional-chaining-tenant-context.md`
- `typed-business-absence.md`
- `dependency-degradation-policy.md`

Each example has:

- bad AI code,
- why it hides failure,
- better semantics,
- review questions.

## Layout

```text
.
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── agents/
│   └── openai.yaml
├── examples/
└── references/
    └── release-checklist.md
```

`references/release-checklist.md` is only for changes that touch release readiness: secrets, auth, external input, deployment, CI, monitoring, rollback, migrations, queues, caches, or external contracts.
