# AI Fallback Disable

> [中文说明](README.zh-CN.md)

Codex skill for removing unsupported fallback behavior from AI-generated code while preserving fallbacks that implement proven policy.

It focuses on one problem: AI often adds fallback values without knowing whether the missing value is optional, invalid, forbidden, unavailable, security-sensitive, or a real business absence.

The operating rule is simple:

> Delete first. Keep only evidence-backed fallback. Add none by default.

The skill treats AI's extra defaults, guards, catches, retries, and compatibility paths as a response to missing context—not as proof that the system needs them. This follows the boundary-versus-core distinction described in [AI 为什么总喜欢写防御性代码？](https://www.nowcoder.com/discuss/886331075914371072?sourceSSR=dynamic): validate genuinely external input at its boundary, then keep core contracts strict and failure semantics visible.

## Removal-First Contract

- Inventory existing fallback before editing.
- Use a default new-fallback budget of zero.
- Remove fallback with the smallest correct change; do not move it into a helper, guard, catch, retry, alternate provider, or degraded result.
- Preserve fallback only when the user or repository provides product, domain, interface, contract-test, or operational evidence.
- If a public contract is genuinely unclear, report the missing policy instead of inventing a temporary fallback.
- Report the final fallback delta: removed, retained, added, and net change.

## Scope

This repo is about fallback audit and failure semantics:

- meaningless fallback such as `""`, `0`, `[]`, `{}`, `"localhost"`, or `"default"` with no policy,
- wrong fallback that invents role, tenant, price, balance, inventory, permission, region, or state,
- swallowed errors such as `catch { return null }` or `catch { return [] }`,
- ambiguous results where not found, forbidden, empty, invalid, and dependency failure collapse into one value,
- unsafe config, secret, environment, and production boundary defaults,
- speculative retries, legacy fields, alternate providers/sources, stale-cache paths, and degraded modes,
- boundary validation that should happen before core business logic.

It is not a grep scanner and it is not a general backend release checklist. Text search can help find smells, but the main job is to judge boundary, contract, and intended failure semantics.

## Examples

The `examples/` folder covers the main cases:

- `meaningless-fallback.md`
- `removal-first-refactor.md`
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
Use $ai-fallback-disable to remove unsupported fallback from this change. Preserve only fallback backed by cited pre-existing policy, add no fallback by default, and report the fallback delta.
```

```text
Use $ai-fallback-disable to clean up this config module removal-first. Required config must fail fast; secrets must not default or auto-trim; low-risk defaults may remain only with cited policy evidence; do not add neighboring defaults for consistency.
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
