# 让 agent 自我进化：dsh-rule-evolve 实战

> 作者：zoahdev · 2026-08-15 · 对应仓库 https://github.com/zoahdev/dsh-rule-evolve（v0.9.0）

## 一句话

把"失败 → 规则 → 验证 → 生效 → 回忆 → 淘汰"变成 agent 自己的循环：每次踩坑都变成经过真实检查验证的规则，装进 profile 的 AGENTS.md，下次会话自动生效。

## 1. 安装

```sh
dsh plugin --profile web add github:zoahdev/dsh-rule-evolve#path:/plugin
```

重启后 agent 拥有四个工具：`evolve_learn`（失败→经验）、`evolve_apply`（验证→安装，未验证拒绝）、`evolve_touch`（使用计数）、`evolve_recall`（本地召回相关规则）。

## 2. CLI 完整闭环

```sh
node scripts/dsh-evolve.mjs extract --task "publish to npm" --from install.log --out experience.jsonl
node scripts/dsh-evolve.mjs verify --experience experience.jsonl --dir ./my-plugin
node scripts/dsh-evolve.mjs evolve --experience experience.jsonl --dir ./my-plugin --profile web
node scripts/dsh-evolve.mjs recall --experience experience.jsonl --query "windows bash"
node scripts/dsh-evolve.mjs score / prune / merge-duplicates
```

## 3. 真实证据（dogfood）

- 从真实 npm ENEEDAUTH 失败日志一键提取 6 条规则；
- `tool-verify` 对 doctor 仓库：8/8 PASS → READY ✅（v1.10.1 前同一道闸抓到过它自己的 lint bug）；
- 规则库 5/5 active、0 重复；"windows bash" 召回 #1856 规则第一；
- 六张 cherry-pick 补丁全部通过官方测试套件验证。

## 4. 为什么这是"自我进化"

规则带来源、验证历史（verifiedCount）、使用历史（usageCount）和生命周期状态（active/stale/retired/merged）。高分规则存活，低分规则被审计性淘汰——不是"记住一切"，而是"记住什么有效"。

## 相关

- 官方讨论：https://github.com/deepseek-ai/deepseek-harness/discussions/1906
- 记忆 RFC：#1881（dsh-rule-evolve 是 Layer 1 规则的生产+校验半层）
