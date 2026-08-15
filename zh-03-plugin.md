# 手把手写第一个 dsh 插件：GitHub Release 雷达

> 作者：zoahdev · 2026-08-15 · 本教程的插件已完整实现并实测，源码见 [dsh-github-release-radar](https://github.com/zoahdev/dsh-github-release-radar)

## 目录

- [为什么做这个插件](#为什么做这个插件)
- [工程结构](#工程结构)
- [package.json：bundle manifest](#packagejsonbundle-manifest)
- [cordis.patch.yml：告诉 dsh 加载谁](#cordispatchyml告诉-dsh-加载谁)
- [插件入口：name / inject / Config / apply](#插件入口name--inject--config--apply)
- [工具定义：github_repo 拆解](#工具定义github_repo-拆解)
- [踩坑：value schema 不支持顶层 required](#踩坑value-schema-不支持顶层-required)
- [HTTP 客户端：取消与限流](#http-客户端取消与限流)
- [测试：mock fetch 的 vitest](#测试mock-fetch-的-vitest)
- [打包、安装、发布](#打包安装发布)

## 为什么做这个插件

DeepSeek Harness 刚开源时社区插件几乎是空白。选 GitHub 数据有三个理由：

1. 官方 REST API 匿名可用（60 次/小时），**不需要用户配 API Key**，试用门槛为零；
2. dsh 用户都是开发者，Release/Star 数据天然有用；
3. 效果可演示：问一句"看看 llama.cpp 最近发布了什么"就有直观输出。

> 设计原则（YAGNI）：只做三个只读工具，不做写操作（提 issue/评论），避免授权和风险问题。

## 工程结构

```text
dsh-github-release-radar/
├── package.json          # 声明 dsh.bundle 与构建脚本
├── cordis.patch.yml      # 插件行（bundle 的配置贡献）
├── tsconfig.json         # strict + NodeNext + declaration
├── vitest.config.ts
├── src/
│   ├── index.ts          # 插件入口 + 3 个工具
│   └── github.ts         # GitHub REST 客户端
├── tests/
│   ├── github.spec.ts    # 客户端测试
│   └── index.spec.ts     # 注册与工具测试
└── README.md             # 中英双语
```

## package.json：bundle manifest

```json
{
  "name": "dsh-github-release-radar",
  "version": "0.1.0",
  "type": "module",
  "main": "lib/index.js",
  "types": "lib/types/index.d.ts",
  "scripts": {
    "build": "tsc -p tsconfig.json",
    "prepare": "pnpm run build",
    "test": "vitest run"
  },
  "peerDependencies": {
    "@deepseek-ai/cordis": "^4.0.1",
    "@deepseek-ai/dsh-tools": "^0.1.0-rc.6"
  },
  "dependencies": {
    "@deepseek-ai/schemastery": "^3.18.1"
  },
  "dsh": {
    "bundle": {
      "patch": "./cordis.patch.yml"
    }
  }
}
```

关键点：

- `dsh.bundle.patch` 声明这个包贡献哪个 patch 层；
- `prepare` 脚本让 **git 安装也能构建**（官方明确要求，git 安装不跑 build 脚本，只跑 prepare）；
- peer 版本务必对齐当前 release train `0.1.0-rc.6`（见[踩坑](#踩坑value-schema-不支持顶层-required)部分）。

## cordis.patch.yml：告诉 dsh 加载谁

```yaml
- insert:
    - id: github-release-radar
      name: dsh-github-release-radar
```

`name` 是 npm 包名（loader 按包名解析 `main`），不是文件路径。用户可以通过更上层的 patch 给这个 id 追加 `config`。

## 插件入口：name / inject / Config / apply

```ts
import type { Context } from '@deepseek-ai/cordis'
import Schema from '@deepseek-ai/schemastery'
import { defineTool } from '@deepseek-ai/dsh-tools'
import { GitHubClient } from './github.js'

export const name = 'github-release-radar'
export const inject = ['tools']

export interface Config {
  githubToken?: string
  timeoutMs?: number
  defaultLimit?: number
  bodyPreviewChars?: number
  userAgent?: string
}

export const Config = Schema.object({
  githubToken: Schema.string(),
  timeoutMs: Schema.number().default(10_000),
  defaultLimit: Schema.number().default(5),
  bodyPreviewChars: Schema.number().default(500),
  userAgent: Schema.string().default('dsh-github-release-radar/0.1.0'),
})

export function apply(ctx: Context, config: Config): void {
  // 校验正整数配置（非法配置加载时大声失败）
  for (const tool of defineTools(config)) {
    ctx.tools.register(tool)
  }
}
```

## 工具定义：github_repo 拆解

```ts
const repo = defineTool({
  name: 'github_repo',
  description:
    'Get the current overview of a public GitHub repository: star count, forks, open issues, '
    + 'primary language, license, description, and last update time.',
  parameters: {
    owner: { type: 'string', required: true, description: 'Repository owner, e.g. deepseek-ai.' },
    repo: { type: 'string', required: true, description: 'Repository name, e.g. deepseek-harness.' },
  },
  output: {
    schema: {
      type: 'object',
      additionalProperties: false,
      properties: {
        fullName: { type: 'string', required: true },
        description: { oneOf: [{ type: 'string' }, { type: 'null' }], required: true },
        stars: { type: 'integer', required: true },
        forks: { type: 'integer', required: true },
        openIssues: { type: 'integer', required: true },
        language: { oneOf: [{ type: 'string' }, { type: 'null' }], required: true },
        license: { oneOf: [{ type: 'string' }, { type: 'null' }], required: true },
        updatedAt: { oneOf: [{ type: 'string' }, { type: 'null' }], required: true },
        htmlUrl: { type: 'string', required: true },
      },
    },
    render: (_args, value) => [{ type: 'text', text: renderRepo(value) }],
  },
  async execute(args, exec) {
    return await client.getRepo(args.owner, args.repo, exec.signal)
  },
  presentCall: (args) => ({
    card: 'generic',
    title: `GitHub repo: ${args.owner}/${args.repo}`,
    kind: 'search',
    rawInput: args,
  }),
})
```

几个设计点：

- `description` 等可空字段用 `oneOf: [{type:'string'}, {type:'null'}]`，让 TS 推断为 `string | null`，与真实 API 数据一致；
- `output.schema` 声明**规范 JSON 值**，`render` 只负责模型可读文本；
- `presentCall` 是纯函数，让 UI 在调用发出时显示卡片；
- `execute` 把 `exec.signal` 透传给 HTTP 客户端，模型取消时请求立即中止。

## 踩坑：value schema 不支持顶层 required

写第一个版本时我在 `output.schema` 里加了 JSON Schema 风格的顶层 `required: [...]` 数组，运行时报错：

```text
JsonSchemaError: unsupported JSON schema: schema.required is not supported by the value schema DSL
```

**dsh 的 value schema DSL 只支持属性级 `required: true`**，不支持对象级 `required` 数组。把顶层 `required` 全部删掉、每个字段保留 `required: true` 即可。

另一个生态级大坑：npm 上 `@deepseek-ai/dsh-tools` 的 latest 指向 `0.0.1-rc.1`，其 peer `@deepseek-ai/dsh-user-approval@0.0.1-rc.1` 依赖**未发布**的 `@deepseek-ai/dsh-type-meta`，独立安装直接 404。务必显式使用 `^0.1.0-rc.6`。

## HTTP 客户端：取消与限流

```ts
private withDeadline(signal: AbortSignal): AbortSignal {
  return AbortSignal.any([signal, AbortSignal.timeout(this.options.timeoutMs)])
}

async getRepo(owner: string, repo: string, signal: AbortSignal): Promise<GitHubRepo> {
  return this.parseRepo(await this.request(`/repos/${encodeURIComponent(owner)}/${encodeURIComponent(repo)}`, signal))
}
```

错误处理要点：

- `403 + x-ratelimit-remaining: 0` → 明确提示"匿名限流，请配置 githubToken"；
- `404` → 提示检查 owner/repo 拼写与仓库是否公开；
- `401` → 提示 token 无效；
- 网络异常区分「被取消」「超时」「网络失败」。

## 测试：mock fetch 的 vitest

```ts
import { afterEach, describe, expect, it, vi } from 'vitest'
import { GitHubClient } from '../src/github.js'

function jsonResponse(body: unknown) {
  return { ok: true, status: 200, headers: new Headers(), json: async () => body } as Response
}

afterEach(() => vi.unstubAllGlobals())

it('maps search results with the requested sort order', async () => {
  const fetchMock = vi.fn(async () => jsonResponse({ items: [/* ... */] }))
  vi.stubGlobal('fetch', fetchMock)

  const hits = await makeClient().searchRepos('agent framework', 5, 'stars', new AbortController().signal)
  expect(fetchMock.mock.calls[0]?.[0]).toBe(
    'https://api.github.com/search/repositories?q=agent%20framework&sort=stars&order=desc&per_page=5',
  )
  expect(hits[0]).toMatchObject({ fullName: 'a/b', stars: 9 })
})
```

用 `vi.stubGlobal('fetch', ...)` 可以完全不联网地测试请求路径、映射和错误分支。本项目共 15 个测试，全部通过。

## 打包、安装、发布

```sh
pnpm run build
pnpm test
pnpm pack    # 生成 dsh-github-release-radar-0.1.0.tgz
```

安装进 web profile：

```sh
dsh plugin --profile web add ./dsh-github-release-radar-0.1.0.tgz
dsh web --port 4099
```

实测：配置层出现在 `dsh --profile web --dump-config` 里，web 启动成功（HTTP 200），无加载错误。

发布三步：

1. 推到 GitHub，仓库加 **`dsh-plugin`** 话题（官方发现机制）；
2. README 写中英双语 + 验证记录（`VERIFICATION.md`）；
3. 可选：`pnpm publish` 发到 npm，用户即可 `dsh plugin add dsh-github-release-radar`。

下一篇：[《在 dsh 生态里被看见：贡献者路线图》](./zh-04-contributing.md)
