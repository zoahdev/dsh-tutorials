# compaction 缓存击穿：为什么每次自动压缩都多付钱

> 作者：zoahdev 路 2026-08-15 路 对应上游补丁 https://github.com/zoahdev/deepseek-harness/tree/fix/compaction-inherit-header-config

## 一句话

压缩请求只继承了 provider/model，却强制加 `maxTokens: 8192`，和正常回合的参数不一致 → provider 前缀缓存全 miss → 每次压缩都把 ~12 万 token 重新计费。

## 1. 现象（真实会话数据）

修复前：

```json
{"inputTokens":122558,"outputTokens":1584}
```

没有 `cacheReadTokens` —— 100% miss。修复后：

```json
{"inputTokens":402,"outputTokens":3548,"cacheReadTokens":220928}
```

99.7% 前缀被缓存复用，只有压缩指令本身是新 token。

## 2. 根因

`summarizeWithLlm`（compaction-basic）构造摘要请求时：

```ts
{ provider, model, messages, system, tools, maxTokens: config.maxTokens(8192), ... }
```

它只从上次路由请求头里挑了 provider/model，丢掉 `reasoningEffort` 等参数，还硬塞一个正常回合没有的 `maxTokens`。provider 的缓存键把 `reasoning_effort`/`max_tokens` 算进去，于是压缩请求看起来像全新请求。

## 3. 修复

`fix/compaction-inherit-header-config`：整包继承路由请求头（与 agent-loop 的 request proposal 同语义），adapter 默认化的字段照常丢弃，不再强制 maxTokens。压缩调用就和正常回合完全同键。

## 4. 自检方法

长会话里看压缩事件的 usage：

- 有 `cacheReadTokens` 且远大于 `inputTokens` → 缓存命中正常；
- `inputTokens` 接近上下文大小、`cacheReadTokens` 为 0 → 缓存被击穿，检查是否用了未打补丁的版本。

## 5. 相关

- 官方讨论：https://github.com/deepseek-ai/deepseek-harness/discussions/1944
- 影响面：DeepSeek 官方 KV cache、OpenAI 风格网关等一切缓存键含请求参数的服务商。
