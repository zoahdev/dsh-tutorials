# dsh-plugin-doctor 排障手册：profile-shadow / manifest-bom / dsh-doctor/v1 契约

> 作者：zoahdev · 2026-08-15 · 对应仓库 https://github.com/zoahdev/dsh-plugin-doctor（v1.6.0）

## 场景

DSH 出问题，先跑一条命令：

```sh
npx dsh-plugin-doctor --profile ~/.dsh/profiles/web --json
```

退出码：`0` 全部通过 / `1` 有警告 / `2` 有 FAIL。

## 检查 1：profile-shadow（#1697 双实例）

**症状**：装任何依赖 `@deepseek-ai/dsh-tools` 的插件后，所有工具调用报
`Cannot read properties of undefined (reading 'prepare')`，会话随后卡死。

**根因**：`nodeLinker: hoisted` 时，插件依赖的 `dsh-tools` 被提升到 profile
顶层 `node_modules`，Node 裸说明符解析命中副本，宿主与副本的
`TOOL_RUNTIME_SCHEDULER` symbol 不是同一个 → `undefined.prepare`。

**诊断输出**：

```json
{ "name": "profile-shadow", "status": "fail",
  "detail": "profile-top-level @deepseek-ai/* real-directory copy(ies) shadow the host: dsh-tools …" }
```

**修复**：官方修复方向是 `Symbol.for` + 协议版本守卫（已备好 cherry-pick
分支）；临时方案是重装 profile 让宿主副本优先解析。诊断消息会直接指路。

## 检查 2：manifest-bom（#1842 启动崩溃）

**症状**：`dsh web` 启动即报 `Unexpected token`，没有任何 BOM 提示。

**根因**：profile 的 `package.json` 带 UTF-8 BOM，`readProfileManifest`
直接 `JSON.parse(readFileSync(..., 'utf8'))` 崩溃。

**诊断输出**：

```json
{ "name": "manifest-bom", "status": "fail",
  "detail": "profile manifest … starts with a UTF-8 BOM; dsh web crashes at boot …" }
```

**修复**：把文件另存为"UTF-8 无 BOM"，或删掉前三个字节
（`EF BB BF`）。上游一行补丁已在 fork 分支 `fix/profile-manifest-bom-strip`。

## 输出契约：dsh-doctor/v1

`--json` 输出统一 envelope：

```json
{
  "schema": "dsh-doctor/v1",
  "profile": "…",
  "exitCode": 2,
  "summary": { "pass": 1, "warn": 0, "fail": 1 },
  "checks": [ { "name": "…", "status": "pass|warn|fail", "detail": "…" } ]
}
```

CI、市场、支持流程都能依赖这套形状。跨实现验收工具：

```sh
node scripts/doctor-contract-check.mjs --impl1 "node lib/bin.js"
```

用 clean / BOM / shadow 三个 fixture 断言退出码与 envelope，双实现可互换。
