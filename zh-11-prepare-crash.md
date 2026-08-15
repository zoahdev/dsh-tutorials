# 调试 dsh 插件崩溃：`undefined.prepare` 全家桶

> 作者：zoahdev · 2026-08-15 · 基于 #1697 / #1763 的真实根因排查与修复

## 症状

```text
Cannot read properties of undefined (reading 'prepare')
```

出现方式很迷惑：有时装完第三方插件后**所有**工具调用都崩；有时只是 `web_search` / `fetch` 等特定工具间歇性崩；重试偶尔能过。DSH 版本 0.1.0-rc.6，Windows/Linux 都可能。

## 根因（一句话版）

`@deepseek-ai/dsh-tools` 用 `Symbol('@deepseek-ai/dsh-tools.scheduler')` 作为内部调度器的 key，而 **unique symbol 的身份是 per-module-instance 的**。当 profile 里出现第二份物理副本（pnpm hoisted 把插件的传递依赖提升到顶层）时，宿主和副本各持一个不同的 symbol——`dsh-agent-loop` 用宿主的 key 去读，读到的却是副本存的值 → `undefined.prepare`。

间歇性是因为只有"命中另一实例调度槽"的那次调用才崩。

## 诊断三步

```sh
# 1. 一键检查 profile 是否有宿主遮蔽（真实目录形式的 @deepseek-ai/* 副本）
node /path/to/dsh-plugin-doctor/lib/bin.js --profile ~/.dsh/profiles/web
# 输出 FAIL + 具体包名即命中

# 2. 手动确认
dir ~/.dsh/profiles/web/node_modules/@deepseek-ai
# 顶层出现 dsh-tools 之类的"真实目录"（不是链接）就是遮蔽

# 3. 确认实际解析版本
pnpm why @deepseek-ai/dsh-tools
```

## 立即修复：link: 依赖

在 profile 的 `package.json` 里把宿主副本以 `link:` 方式钉住，让 profile 与宿主共用同一份 dsh-tools（同一 realpath = 同一模块实例 = symbol 一致）：

```jsonc
{
  "dependencies": {
    "@deepseek-ai/dsh-tools": "link:C:/Users/<user>/AppData/Roaming/npm/node_modules/@deepseek-ai/dsh/node_modules/@deepseek-ai/dsh-tools"
  }
}
```

然后在 profile 目录 `pnpm install`，再试工具调用。路径按你机器上的 npm 全局前缀调整。

## 上游修复（已备好，等 PR 通道开放）

https://github.com/zoahdev/deepseek-harness/tree/fix/tool-runtime-scheduler-symbol-for

两层：

1. `Symbol.for('@deepseek-ai/dsh-tools.scheduler')`——同一进程所有副本共享 key；
2. `TOOL_RUNTIME_SCHEDULER_PROTOCOL_VERSION` + `assertSchedulerProtocol()`——跨版本副本仍然响亮报错，不会静默跑错协议。

已用 npm 上真实的 rc.6 包验证：修复前两个物理副本 symbol 不相等，修复后相等。

## 别混淆：allowBuilds 拦截是另一件事

git 安装插件时如果报 `ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED`，那是 pnpm 11 禁止 git 依赖执行 `prepare`——不是上面的运行时崩溃。解法：把 dsh CLI 打印的 key 加进 profile 的 `pnpm-workspace.yaml` 的 `allowBuilds`，再跑一次安装。

## 预防

- 发布前：`dsh-plugin-doctor --full .`（真实装进全新 profile 验证）；
- 模板自带运行时守卫：dsh-tools 版本不匹配时 `apply()` 直接抛带指引的错误（[dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)）；
- 遇到崩溃先跑 `--profile`，别急着重装。

## 相关讨论

- #1697（根因 + 修复）：https://github.com/deepseek-ai/deepseek-harness/discussions/1697
- #1763（同一崩溃新报）：https://github.com/deepseek-ai/deepseek-harness/discussions/1763
