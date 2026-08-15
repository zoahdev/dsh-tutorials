# Build Your First dsh Plugin: GitHub Release Radar

> By zoahdev · 2026-08-15 · The plugin in this tutorial is fully implemented and tested — see [dsh-github-release-radar](https://github.com/zoahdev/dsh-github-release-radar)

## Table of Contents

- [Why this plugin](#why-this-plugin)
- [Project structure](#project-structure)
- [package.json: the bundle manifest](#packagejson-the-bundle-manifest)
- [cordis.patch.yml: tell dsh what to load](#cordispatchyml-tell-dsh-what-to-load)
- [Plugin entry: name / inject / Config / apply](#plugin-entry-name--inject--config--apply)
- [Tool definition: github_repo walkthrough](#tool-definition-github_repo-walkthrough)
- [Pitfall: no top-level required in value schemas](#pitfall-no-top-level-required-in-value-schemas)
- [HTTP client: cancellation and rate limits](#http-client-cancellation-and-rate-limits)
- [Testing: vitest with mocked fetch](#testing-vitest-with-mocked-fetch)
- [Pack, install, publish](#pack-install-publish)

## Why this plugin

When DeepSeek Harness was open-sourced, the community plugin ecosystem was nearly empty. GitHub data was the obvious first pick:

1. The public REST API works anonymously (60 requests/hour) — **no API key needed**, zero friction to try;
2. dsh users are developers; release/star data is genuinely useful;
3. It demos beautifully: "What did llama.cpp release recently?"

> Design rule (YAGNI): three read-only tools only. No write operations (issues/comments) to avoid authorization and risk.

## Project structure

```text
dsh-github-release-radar/
├── package.json          # dsh.bundle manifest + build scripts
├── cordis.patch.yml      # the plugin row (the bundle's config contribution)
├── tsconfig.json         # strict + NodeNext + declaration
├── vitest.config.ts
├── src/
│   ├── index.ts          # plugin entry + 3 tools
│   └── github.ts         # GitHub REST client
├── tests/
│   ├── github.spec.ts
│   └── index.spec.ts
└── README.md             # bilingual
```

## package.json: the bundle manifest

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

Key points:

- `dsh.bundle.patch` declares which patch layer this package contributes;
- `prepare` builds from source so **git installs work** (official requirement — git installs never run `build`, only `prepare`);
- Pin peers to the current release train `0.1.0-rc.6` (see the pitfall below).

## cordis.patch.yml: tell dsh what to load

```yaml
- insert:
    - id: github-release-radar
      name: dsh-github-release-radar
```

`name` is the npm package name (the loader resolves `main`), not a file path. Users can add `config` to this id from a higher patch layer.

## Plugin entry: name / inject / Config / apply

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
  for (const tool of defineTools(config)) {
    ctx.tools.register(tool)
  }
}
```

## Tool definition: github_repo walkthrough

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

Design notes:

- Nullable fields use `oneOf: [{type:'string'}, {type:'null'}]` so TypeScript infers `string | null`, matching real API data;
- `output.schema` declares the **canonical JSON value**; `render` owns model-facing prose;
- `presentCall` is a pure function that gives the UI a card when the call is made;
- `execute` passes `exec.signal` into the HTTP client so cancellation aborts in-flight requests.

## Pitfall: no top-level required in value schemas

My first version used JSON-Schema-style top-level `required: [...]` arrays and failed at runtime:

```text
JsonSchemaError: unsupported JSON schema: schema.required is not supported by the value schema DSL
```

**The dsh value schema DSL only supports per-property `required: true`** — no object-level `required` array. Remove the top-level arrays and keep `required: true` on each property.

Ecosystem-level pitfall: npm's `latest` tag for `@deepseek-ai/dsh-tools` is `0.0.1-rc.1`, whose peer `@deepseek-ai/dsh-user-approval@0.0.1-rc.1` depends on the unpublished `@deepseek-ai/dsh-type-meta` — standalone installs 404. Always pin `^0.1.0-rc.6`.

## HTTP client: cancellation and rate limits

```ts
private withDeadline(signal: AbortSignal): AbortSignal {
  return AbortSignal.any([signal, AbortSignal.timeout(this.options.timeoutMs)])
}

async getRepo(owner: string, repo: string, signal: AbortSignal): Promise<GitHubRepo> {
  return this.parseRepo(await this.request(`/repos/${encodeURIComponent(owner)}/${encodeURIComponent(repo)}`, signal))
}
```

Error handling:

- `403 + x-ratelimit-remaining: 0` → "anonymous rate limit exceeded; set githubToken";
- `404` → check owner/repo spelling and public visibility;
- `401` → invalid token;
- network failures distinguish cancelled / timed out / failed.

## Testing: vitest with mocked fetch

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

`vi.stubGlobal('fetch', ...)` tests request paths, mapping, and error branches with zero network. This project ships 15 passing tests.

## Pack, install, publish

```sh
pnpm run build
pnpm test
pnpm pack    # → dsh-github-release-radar-0.1.0.tgz
```

Install into the web profile:

```sh
dsh plugin --profile web add ./dsh-github-release-radar-0.1.0.tgz
dsh web --port 4099
```

Verified: the layer shows up in `dsh --profile web --dump-config`, and the web app boots with HTTP 200 and no plugin errors.

Publishing checklist:

1. Push to GitHub and add the **`dsh-plugin`** topic (official discovery mechanism);
2. Bilingual README + a `VERIFICATION.md` record;
3. Optionally `pnpm publish`, so users can `dsh plugin add dsh-github-release-radar`.

Next: [Getting Noticed in the dsh Ecosystem: Contributor Roadmap](./en-04-contributing.md)
