# AI Fallback Disable

> [English README](README.md)

这是一个清理 AI 生成代码里无依据 fallback、同时保留真正策略性 fallback 的 Codex skill。

它聚焦一个问题：AI 经常乱加 fallback，但它并不知道这里到底是可选缺省、非法输入、无权限、依赖失败、安全边界，还是正常的业务空结果。

执行规则很简单：

> 删除优先；只保留有证据的 fallback；默认不新增。

AI 常因上下文不完整而追加默认值、guard、`try/catch`、重试和兼容分支，但“看起来更稳”不是系统需要它们的证据。这个 skill 采用[《AI 为什么总喜欢写防御性代码？》](https://www.nowcoder.com/discuss/886331075914371072?sourceSSR=dynamic)里的核心区分：真实外部输入在边界校验，核心逻辑保持严格契约，失败语义继续可见。

## 删除优先契约

- 修改前先盘点现有 fallback。
- 默认新增 fallback 预算为零。
- 用最小改动删除无依据 fallback，不能把它迁移进 helper、guard、catch、retry、备用 provider 或 degraded result。
- 只有用户要求或仓库内的产品、领域、接口、契约测试、运维策略能够举证时，才保留 fallback。
- 公共契约确实不清楚时，明确报告缺少的策略决定，不能猜一个临时兜底。
- 最后报告 fallback delta：删除、保留、新增和净变化。

## 范围

这个仓库管的是 fallback 审计和失败语义：

- 无意义 fallback：`""`、`0`、`[]`、`{}`、`"localhost"`、`"default"` 这类没有业务策略的占位值。
- 错误 fallback：伪造 role、tenant、price、balance、inventory、permission、region、state 等业务事实。
- 吞错 fallback：`catch { return null }`、`catch { return [] }` 这类把系统异常变成正常结果的写法。
- 语义混淆：not found、forbidden、empty、invalid、dependency failure 全都返回同一个值。
- 配置、secret、环境、生产边界上的危险默认值。
- 无依据的重试、旧字段、备用 provider/数据源、陈旧缓存和降级模式。
- 本该在边界完成的校验，却散落到核心业务逻辑里。

它不是 grep 扫描器，也不是通用后端发布清单。文本搜索只能当辅助，核心是判断 boundary、contract 和 failure semantics。

## 例子

`examples/` 里是几个主要场景：

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

每个例子都说明坏代码藏住了什么，以及更合理的语义是什么。

## 用法

```text
Use $ai-fallback-disable to remove unsupported fallback from this change. Preserve only fallback backed by cited pre-existing policy, add no fallback by default, and report the fallback delta.
```

```text
Use $ai-fallback-disable to clean up this config module removal-first. Required config must fail fast; secrets must not default or auto-trim; low-risk defaults may remain only with cited policy evidence; do not add neighboring defaults for consistency.
```

## 目录

```text
.
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── agents/
│   └── openai.yaml
└── examples/
```
