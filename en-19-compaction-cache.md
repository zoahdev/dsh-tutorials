# Compaction cache misses: why every auto-compaction re-bills the whole context

> By zoahdev 路 2026-08-15 路 Upstream patch: https://github.com/zoahdev/deepseek-harness/tree/fix/compaction-inherit-header-config

## TL;DR

The summarizer request only inherited provider/model and forced `maxTokens: 8192`, so its parameters diverged from normal turns - the provider prefix cache missed completely and ~122k tokens were re-billed on every compaction.

## 1. Evidence (real session data)

Before the fix:

```json
{"inputTokens":122558,"outputTokens":1584}
```

No `cacheReadTokens` - a 100% miss. After the fix:

```json
{"inputTokens":402,"outputTokens":3548,"cacheReadTokens":220928}
```

99.7% of the prefix is reused; only the compaction instruction itself is new.

## 2. Root cause

`summarizeWithLlm` (compaction-basic) built the summarizer call as:

```ts
{ provider, model, messages, system, tools, maxTokens: config.maxTokens(8192), ... }
```

It picked only provider/model from the last routed request header, dropped `reasoningEffort` and friends, and forced a `maxTokens` that normal turns never send. Providers whose cache key includes `reasoning_effort` / `max_tokens` (DeepSeek KV cache, OpenAI-style gateways) treated the compaction call as brand new.

## 3. Fix

`fix/compaction-inherit-header-config`: inherit the routed header config wholesale (same semantics as agent-loop's request proposal), drop adapter-defaulted fields as usual, and stop forcing maxTokens. The compaction call now keeps the exact cache key of normal turns.

## 4. How to self-check

In a long session, look at the compaction event usage:

- `cacheReadTokens` present and much larger than `inputTokens` -> cache is being reused;
- `inputTokens` near the context size with `cacheReadTokens: 0` -> the cache is being defeated; check whether you run a patched build.

## 5. References

- Official discussion: https://github.com/deepseek-ai/deepseek-harness/discussions/1944
- Affects every provider/gateway whose cache key includes request parameters (DeepSeek KV cache, OpenAI-style gateways).
