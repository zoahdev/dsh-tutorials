# 进化徽章：把规则库变成 README 上的 SVG

> 作者：zoahdev 路 2026-08-15 路 对应 dsh-rule-evolve v0.11.0

## 一句话

`dsh-evolve badge` 一条命令，把"我的 agent 学到了多少条已验证规则"变成可嵌入 README 的 SVG 徽章。

## 1. 用法

```sh
dsh-evolve badge --experience experience.jsonl --evolution EVOLUTION.md --out badge.svg
```

```markdown
![agent rules](badge.svg)
```

## 2. 等级

按已验证规则数分档：`starting`（0）→ `learning`（1-4）→ `building`（5-9）→ `growing`（10-19）→ `legend`（20+），颜色跟着变。

## 3. CI 友好

`--json` 输出元数据 + SVG 载荷，发布流水线可以直接把徽章写进 README：

```sh
dsh-evolve badge --experience experience.jsonl --json > badge.json
```

## 4. 为什么值得

自我进化的价值在"被看见"时才成立。成长报告是长文，分享卡是图片，徽章是**常驻门面**——挂在 README 第一行，每个访客都看到你的 agent 在成长。
