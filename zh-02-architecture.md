# 理解 dsh 架构：一切皆插件与 Cordis

> 作者：zoahdev · 2026-08-15 · 基于官方仓库源码与文档

## 目录

- [一句话架构](#一句话架构)
- [插件是什么](#插件是什么)
- [Context 与 apply](#context-与-apply)
- [依赖注入：inject](#依赖注入inject)
- [效果即注册：ctx.effect](#效果即注册ctxeffect)
- [工具 DSL：defineTool](#工具-dsldefinetool)
- [配置：Schemastery](#配置schemastery)
- [组合机制：bundle、profile、patch](#组合机制bundleprofilepatch)
- [官方仓库布局](#官方仓库布局)
- [与 MCP 的关系](#与-mcp-的关系)
- [关键术语表](#关键术语表)

## 一句话架构

> dsh = Cordis（插件运行时）+ 能力包（tools/llm/fs/shell/web/…）+ 会话层（事件溯源）+ 组合机制（profile/patch）

每个能力都是一组插件；插件通过注册表和事件互相协作；agent 循环（`agent-loop`）把模型、工具、会话串起来。官方把这种设计称为「一切皆插件」：**官方包并不比社区包更重要**。

## 插件是什么

在 dsh 里，插件就是一个导出 `name` 和 `apply(ctx)` 的 TypeScript/JavaScript 模块：

```ts
import type { Context } from '@deepseek-ai/cordis'

export const name = 'my-plugin'

export function apply(ctx: Context) {
  // 在这里注册能力
}
```

三种形式：

| 形式 | 适用场景 |
|---|---|
| 函数式（`export function apply`） | 大多数情况，最简单 |
| 对象式（`export default { name, apply }`） | 需要组织多个字段 |
| 类式（`extends Service`） | 需要向其他插件提供**服务**（service） |

## Context 与 apply

`apply(ctx, config)` 在插件加载时被调用一次，`ctx` 是上下文对象：

- `ctx.on(...)` / `ctx.effect(...)`：注册事件监听和副作用；
- `ctx.inject([...], (nestedCtx) => {})`：声明式地等待服务就绪；
- `ctx.tools`、`ctx.web`、`ctx.llm` 等：其他插件提供的服务（依赖注入后可用）。

## 依赖注入：inject

插件需要别的服务时，用 `inject` 声明；Cordis 保证依赖就绪后才调用 `apply`：

```ts
export const name = 'my-tool-plugin'
export const inject = ['tools']

export function apply(ctx: Context) {
  ctx.tools.register(/* ... */)
}
```

官方约定：**注册都是 effect**——`ctx.tools.register(...)` 返回 disposer，插件卸载时自动清理，不需要手动 removeListener。

## 效果即注册：ctx.effect

有需要手动清理的资源（定时器、网络连接）时：

```ts
export function apply(ctx: Context) {
  ctx.effect(() => {
    const timer = setInterval(() => console.log('heartbeat'), 5000)
    return () => clearInterval(timer)
  })
}
```

配置热更新时，旧实例的注册全部自动卸载，新配置重新加载。

## 工具 DSL：defineTool

给模型用的工具用 `@deepseek-ai/dsh-tools` 的 `defineTool` 定义：

```ts
import { defineTool } from '@deepseek-ai/dsh-tools'

ctx.tools.register(defineTool({
  name: 'greet',
  description: 'Greet someone by name.',
  parameters: {
    name: { type: 'string', required: true, description: 'The name to greet' },
  },
  output: {
    schema: { type: 'string' },
    render: (_args, value) => [{ type: 'text', text: value }],
  },
  async execute(args, exec) {
    return `Hello, ${args.name}!`
  },
}))
```

要点：

- `parameters`/`output.schema` 是作者面 schema，自动编译成模型看到的 JSON Schema，并**在 execute 前校验参数**；
- `output.render` 是纯函数，把规范值（canonical value）转成模型可读文本；
- `presentCall`/`presentResult` 决定 UI 卡片（generic/terminal/diff/search/web）；
- `execute(args, exec)` 里必须尊重 `exec.signal`（取消信号），长任务要考虑后台任务注册；
- 返回一个规范的 JSON 值，不要返回给人类看的散文（那是 render 的活）。

## 配置：Schemastery

插件通过同名 `Config` schema 接收配置：

```ts
import Schema from '@deepseek-ai/schemastery'

export interface Config {
  greeting: string
  maxRetries: number
}

export const Config = Schema.object({
  greeting: Schema.string().default('Hello'),
  maxRetries: Schema.number().default(3),
})
```

原则：部署环境可能不同的值（超时、条数、开关）都必须是配置项，不能硬编码；非法配置在加载时大声失败。

## 组合机制：bundle、profile、patch

三个概念：

1. **bundle（组合包）**：一个带 `dsh.bundle` manifest 的 npm 包，贡献一段 `cordis.patch.yml`（插件行）。这是插件作者分发的东西。
2. **profile**：`$DSH_HOME/profiles/<name>` 下的一份可启动组合，按有序 `bundles` 列表叠加。
3. **patch（覆盖层）**：YAML patch 行数组，后加的层按 `id` 覆盖先前的行。

层顺序（后层胜出）：

```text
profile.bundles → profile 自身 patch → $DSH_HOME 级 patch → 命令行 --patch
```

## 官方仓库布局

```text
vendor/      Cordis 源码（vendored，带上游 SHA）
packages/    @deepseek-ai/dsh-* workspace 包
  core/       会话、system-prompt、tools、agent、agent-loop
  api/        BFF / RPC gateway
  llm/        LLM 能力（DeepSeek provider 等）
  fs/ shell/ web/ lsp/ subprocess/  各种能力包
  todo/ plan/ goal/ skill/ workflow/ 业务插件
  bundle/     可安装的 dsh --profile patch 层
examples/    可运行的 cordis.yml 示例（如 web-schedule）
docs/        架构、目录、cookbook
```

## 与 MCP 的关系

dsh 有两类工具来源：

- **原生工具**：用 `defineTool` 直接注册（todo、fs、bash、web…）；
- **MCP 工具**：通过 `@deepseek-ai/dsh-mcp-client` 连接外部 MCP 服务器，把远程工具映射进注册表。

所以「写 dsh 插件」和「接 MCP」是互补的：MCP 适合复用现成生态，原生插件适合深度定制（UI 卡片、策略、会话状态）。

## 关键术语表

| 术语 | 含义 |
|---|---|
| harness | 让 agent 跑起来的框架层（任务分解、工具、会话） |
| Context | Cordis 上下文，插件的注册入口 |
| inject | 声明依赖，Cordis 保证就绪顺序 |
| effect | 有清理函数的副作用注册 |
| canonical value | 工具返回的规范 JSON 值（模型与程序共用） |
| render intent | UI 卡片声明（generic/terminal/diff/search/web） |
| bundle / profile / patch | 分发单元 / 启动组合 / 覆盖层 |
| schema | 作者面 DSL（parameters/output/Config），自动生成 JSON Schema |

下一篇：[《手把手写第一个 dsh 插件》](./zh-03-plugin.md)
