# dsh-pet-evolve：把 agent 成长变成一只宠物

> 作者：zoahdev 路 2026-08-15 路 仓库 https://github.com/zoahdev/dsh-pet-evolve（v0.2.0）

## 一句话

宠物图鉴给你看贴图；dsh-pet-evolve 的每个形态都是 **agent 真实信号挣来的**：已验证规则、完成会话、工具调用、压缩摘要，全部折成 XP。

## 1. 进化

蛋 → 幼崽 → 少年 → 成年 → 传说（300 / 800 / 1600 / 3000 XP）。每 100 XP 升 1 级；形态是 Canvas 实时绘制的，四款皮肤：鲸鱼 / 猫 / 机器人 / 幽灵。

## 2. 真实信号

```sh
npx dsh-pet-evolve --profile ~/.dsh/profiles/web
```

- `rule_verified`：规则库中已验证的规则（兼容 dsh-rule-evolve）
- `session_completed`：完成的会话
- `tool_call`：会话日志里的工具调用
- `compaction`：压缩摘要

宠物还会镜像 agent 状态：working 时蹦跶+齿轮、done 时彩带、failed 时难过、idle 时眨眼。

## 3. 分享

一键导出 1200×630 成长卡 PNG："My agent pet reached Legend · Level 12 · 25 rules verified"。

## 4. DSH 插件模式

```sh
dsh plugin --profile web add github:zoahdev/dsh-pet-evolve
```

`lib/` 已提交，安装免构建（#1965 的教训直接落地）。

## 5. 隐私

只读你指定的本地文件，零遥测，分享卡在浏览器本地生成。
