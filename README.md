# AI Fallback Disable

> [中文说明](README.zh-CN.md)

Codex skill for reviewing AI-generated environment/config loading code.

It focuses on one problem: AI often adds fallback values to config code without knowing whether the value is optional, required, sensitive, or safe to default.

The rule is simple:

> Meaningless fallback should not exist in code.

## Scope

This repo is about config loading:

- scattered `process.env` reads,
- `process.env.X || default`,
- unsafe `trim()`,
- secret defaults,
- loose number/boolean parsing,
- startup validation,
- typed config objects.

It is not a general backend error-handling skill.

## Examples

The `examples/` folder covers the main cases:

- `bad-env-loader.md`
- `meaningless-fallback.md`
- `secret-trim.md`
- `default-policy.md`
- `zod-env-schema.md`

Each example explains the bad code, what it hides, and the better config semantics.

## Usage

```text
Use $ai-fallback-disable to review this config loader. Focus on meaningless fallback, unsafe defaults, and secret handling.
```

```text
Use $ai-fallback-disable to write a startup config module. Required config must fail fast; secrets must not default or auto-trim.
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
