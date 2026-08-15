# 让 agent 的成长看得见：Agent Growth Report 实战

> 作者：zoahdev · 2026-08-15 · 对应仓库 https://github.com/zoahdev/dsh-rule-evolve（v0.10.2）

## 一句话

一条命令，把 agent 的规则库和进化历史变成一张漂亮的成长报告（HTML + 可下载 PNG 分享卡）——"我的 agent 学到了 N 条经验，全部通过真实检查验证"。

## 1. 生成报告

```sh
node scripts/dsh-evolve.mjs dash --experience experience.jsonl --evolution EVOLUTION.md --out agent-growth.html
```

生成单文件 HTML（零依赖、离线可开、中英切换）：

- 规则增长曲线（按日期累计）
- 规则库健康度（active / stale / retired / merged）
- "agent 学到了什么"（主题标签）
- 最常被使用的规则
- 进化时间线（Round 1/2/3…）
- **Agent Growth Report 分享卡** + 一键导出 PNG

## 2. 会话内版本

装 dsh-rule-evolve 插件后，agent 自己就能汇报：

```text
evolve_report(profile: "web")
→ "🐋 My DeepSeek Harness agent learned 11 rule(s), 11/11 verified by real checks, used 3 time(s)…"
```

## 3. 真实示例

马拉松成长日记的可视化：https://github.com/zoahdev/dsh-rule-evolve/blob/main/examples/marathon/agent-growth.html

## 4. 为什么值得做

自我进化的价值只有在被看见时才被认可。成长报告把"验证驱动的学习"变成可分享的成果——发给同事、贴进周报、发到社区，都只需要一张图。
