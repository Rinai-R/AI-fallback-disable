# AI Fallback Disable

> [English README](README.md)

这是一个用来审 AI 生成 fallback 代码的 Codex skill。

它处理的不是普通代码风格，而是那种“看起来很稳，其实把错误藏起来了”的代码：配置错了没炸、secret 缺了还跑、数据库挂了返回空数组、租户上下文没了还继续查、多个错误都用 `null` 表示。

## 它看什么

- 不安全的 fallback。
- 被吞掉的错误。
- 散落的配置读取。
- secret 默认值，或者对 secret 静默清洗。
- `null`、空数组、空对象、`Optional.empty()` 被拿来表达多种含义。
- optional chaining 把本该存在的领域约束藏掉。

核心问题就一个：

> 这是正常业务结果，还是代码把失败伪装成了结果？

## 它不是什么

它不是 grep 扫描器，也不是通用代码风格审查。

文本搜索只能当辅助雷达。真正要判断的是边界、契约、失败类型、调用方预期和可观测性。

## 用法

```text
Use $ai-fallback-disable to review this config loader. Focus on unsafe defaults, secrets, and failure semantics.
```

```text
Use $ai-fallback-disable to review this handler. Focus on swallowed errors and ambiguous empty returns.
```

## 例子

`examples/` 里放了几个短案例：

- `env-config.md`
- `swallowed-db-error.md`
- `optional-chaining-tenant-context.md`
- `typed-business-absence.md`
- `dependency-degradation-policy.md`

每个例子都按这个结构写：

- 坏的 AI 代码。
- 它为什么隐藏失败。
- 更好的失败语义。
- 审查时该问什么。

## 目录

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

`references/release-checklist.md` 只在变更影响发布准备时使用，比如 secret、鉴权、外部输入、部署、CI、监控、回滚、迁移、队列、缓存或外部 contract。
