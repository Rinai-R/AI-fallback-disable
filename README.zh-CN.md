# AI Fallback Disable

> [English README](README.md)

`ai-fallback-disable` 是一个 Codex skill，用来审查和加固 AI 生成代码中常见的“假稳健”问题：不安全 fallback、吞错、配置默认值、secret 兜底、模糊空返回，以及削弱领域契约的 optional chaining。

核心原则：

> 不要通过隐藏失败来让代码“安全”。

真正可靠的代码应该在边界显式校验输入和配置，在核心逻辑中保持清晰契约，并让每个 fallback 都有明确的业务或运维语义。

## 适用场景

使用这个 skill 处理：

- AI 生成代码里到处出现 `fallback`、默认值、空数组、空对象、`null` 返回。
- `try/catch` 把未知错误吞掉，或者把依赖失败伪装成成功结果。
- `process.env.X || default`、`Number(value) || default` 混淆缺失、非法值和合法 falsy。
- `JWT_SECRET`、API key、token、签名密钥等 secret 出现默认值或被静默 `trim()`。
- `user?.id`、`tenant?.id`、`permission?.role` 这类 required context 被 optional chaining 掩盖。
- 想把“让代码更 robust”改成明确的失败语义、测试和 verifier 检查。

## 不适用场景

这个 skill 不是通用代码审查清单，也不是发布治理总纲。它不应该在所有代码优化任务中触发。

不要用它来：

- 审查普通代码风格问题。
- 机械删除所有 fallback。
- 把所有业务空值都改成异常。
- 默认生成 grep 脚本或 lint 规则。
- 替代项目已有的安全、发布或架构规范。

## 使用方式

在支持 skill 的 Codex 环境中，可以显式调用：

```text
Use $ai-fallback-disable to review this AI-generated config loader for unsafe fallback and unclear failure semantics.
```

或者：

```text
Use $ai-fallback-disable to harden this API handler. Focus on swallowed errors, ambiguous empty returns, and tenant/auth contract violations.
```

## 工作方式

skill 会引导 agent：

1. 找到相关边界：配置、请求输入、外部 API、文件、数据库/缓存/队列响应、身份/租户/权限上下文。
2. 说明边界校验后核心逻辑应满足的契约。
3. 分类可疑 fallback、默认值、catch、optional chaining 或空返回。
4. 判断正确行为：启动失败、validation error、typed business result、propagated system error 或 explicit degraded mode。
5. 做最小安全改动，不破坏合法业务空值。
6. 补充聚焦的 failure-semantics 测试。

## 仓库结构

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

- `SKILL.md`：核心 skill 指令。
- `agents/openai.yaml`：展示名、短描述和默认 prompt。
- `references/release-checklist.md`：仅当变更触及生产发布、secret、auth、外部输入、部署、CI、监控、回滚等场景时使用。

## 设计取舍

这个 skill 有意避免“默认上脚本/grep”。自动化只适合重复、低误报、稳定且风险足够高的问题。普通审查应先理解边界、契约和失败语义，再决定是否需要 lint、script 或 verifier gate。
