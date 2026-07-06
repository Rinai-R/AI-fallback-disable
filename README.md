# AI Fallback Disable

> [中文说明](README.zh-CN.md)

Codex skill for reviewing fallback semantics in AI-generated code.

It focuses on one problem: AI often adds fallback values without knowing whether the missing value is optional, invalid, forbidden, unavailable, security-sensitive, or a real business absence.

The rule is simple:

> Meaningless fallback should not exist in code.

The broader rule is just as important: wrong fallback should not turn an error into a fake business fact.

## Scope

This repo is about fallback audit and failure semantics:

- meaningless fallback such as `""`, `0`, `[]`, `{}`, `"localhost"`, or `"default"` with no policy,
- wrong fallback that invents role, tenant, price, balance, inventory, permission, region, or state,
- swallowed errors such as `catch { return null }` or `catch { return [] }`,
- ambiguous results where not found, forbidden, empty, invalid, and dependency failure collapse into one value,
- unsafe config, secret, environment, and production boundary defaults,
- boundary validation that should happen before core business logic.

It is not a grep scanner and it is not a general backend release checklist. Text search can help find smells, but the main job is to judge boundary, contract, and intended failure semantics.

## Examples

The `examples/` folder covers the main cases:

- `meaningless-fallback.md`
- `wrong-fallback.md`
- `swallowed-error-fallback.md`
- `semantic-confusion.md`
- `valid-fallback.md`
- `bad-env-loader.md`
- `secret-trim.md`
- `default-policy.md`
- `zod-env-schema.md`

Each example explains the bad code, what it hides, and the better semantics.

## Usage

```text
Use $ai-fallback-disable to review this change for meaningless fallback, wrong fallback, swallowed errors, and ambiguous empty/null results.
```

```text
Use $ai-fallback-disable to review this config module. Required config must fail fast; secrets must not default or auto-trim.
```

## Layout

```text
.
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── agents/
│   └── openai.yaml
└── examples/
```
