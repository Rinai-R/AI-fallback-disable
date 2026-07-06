# AI Fallback Disable

> [English README](README.md)

这是一个审 AI 生成环境变量/配置加载代码的 Codex skill。

它只聚焦一个问题：AI 经常在配置代码里乱加 fallback，但它并不知道这个值是可选、必填、敏感，还是可以安全默认。

规则很简单：

> 无意义的 fallback 不应该存在于代码里。

## 范围

这个仓库只管配置加载：

- 散落的 `process.env` 读取。
- `process.env.X || default`。
- 不该有的 `trim()`。
- secret 默认值。
- 数字/布尔值的宽松解析。
- 启动期校验。
- 类型明确的 config 对象。

它不是通用后端错误处理规范。

## 例子

`examples/` 里是几个主要场景：

- `bad-env-loader.md`
- `meaningless-fallback.md`
- `secret-trim.md`
- `default-policy.md`
- `zod-env-schema.md`

每个例子都说明坏代码藏住了什么，以及更合理的配置语义是什么。

## 用法

```text
Use $ai-fallback-disable to review this config loader. Focus on meaningless fallback, unsafe defaults, and secret handling.
```

```text
Use $ai-fallback-disable to write a startup config module. Required config must fail fast; secrets must not default or auto-trim.
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
