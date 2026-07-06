# AI Fallback Disable

> [English README](README.md)

这是一个审 AI 生成代码里 fallback 语义的 Codex skill。

它聚焦一个问题：AI 经常乱加 fallback，但它并不知道这里到底是可选缺省、非法输入、无权限、依赖失败、安全边界，还是正常的业务空结果。

规则很简单：

> 无意义的 fallback 不应该存在于代码里。

再进一步：错误 fallback 也不应该把真实错误伪造成业务事实。

## 范围

这个仓库管的是 fallback 审计和失败语义：

- 无意义 fallback：`""`、`0`、`[]`、`{}`、`"localhost"`、`"default"` 这类没有业务策略的占位值。
- 错误 fallback：伪造 role、tenant、price、balance、inventory、permission、region、state 等业务事实。
- 吞错 fallback：`catch { return null }`、`catch { return [] }` 这类把系统异常变成正常结果的写法。
- 语义混淆：not found、forbidden、empty、invalid、dependency failure 全都返回同一个值。
- 配置、secret、环境、生产边界上的危险默认值。
- 本该在边界完成的校验，却散落到核心业务逻辑里。

它不是 grep 扫描器，也不是通用后端发布清单。文本搜索只能当辅助，核心是判断 boundary、contract 和 failure semantics。

## 例子

`examples/` 里是几个主要场景：

- `meaningless-fallback.md`
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
Use $ai-fallback-disable to review this change for meaningless fallback, wrong fallback, swallowed errors, and ambiguous empty/null results.
```

```text
Use $ai-fallback-disable to review this config module. Required config must fail fast; secrets must not default or auto-trim.
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
