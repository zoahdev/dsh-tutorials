# The DeepSeek Harness Plugin Pipeline: 8 verified plugins in one day

> by zoahdev · 2026-08-18 · every plugin live on npm/GitHub with green CI (verified on Windows Node 24 / pnpm 11 / dsh 0.1.0-rc.6)

# The DeepSeek Harness Plugin Pipeline: 8 verified plugins in one day

> by zoahdev · 2026-08-18 · every plugin is live on npm/GitHub with green CI

## TL;DR

> A dsh plugin = one `defineTool` + one `cordis.patch.yml` + tests/CI + a bilingual README. Once the quality gate is a repeatable pipeline, going from idea to published plugin takes tens of minutes.

## The minimal skeleton

```
my-plugin/
├── package.json          # name/version/description + dsh.bundle.patch
├── cordis.patch.yml      # - insert: - id: xxx  name: my-plugin
├── src/index.ts          # apply(ctx, config) → ctx.tools.register(defineTool({...}))
├── src/version.ts        # zero-dep peer guard (pnpm can silently link an old RC)
├── tests/*.spec.ts       # vitest, incl. mock-registry / mock-exec end-to-end cases
├── scripts/integration-test.mjs  # pack → fresh install → call the real handler → assert render
├── scripts/dsh-smoke.sh  # fresh profile → plugin add → dump-config → dsh web HTTP 200
└── .github/workflows/ci.yml
```

Gotchas learned from dsh-tools' typings:

- `defineTool` needs `parameters`, `output.schema` (do NOT put `required` in an output object — the value schema DSL rejects it), a pure `render`, and `execute` returning canonical JSON while honoring `exec.signal`.
- Return types that flow into `JsonValue` must be `type` aliases, not `interface`s (index-signature mismatch).
- Peer guard: at apply() time, resolve the linked `@deepseek-ai/dsh-tools` version and throw on mismatch — turn a silent pnpm link error into a loud, actionable one.

## The quality gate (every plugin runs this)

```sh
pnpm install && pnpm typecheck && pnpm build && pnpm test
pnpm pack
node scripts/integration-test.mjs ./x-0.1.0.tgz   # real tarball → real handler → real render
bash scripts/dsh-smoke.sh ./x-0.1.0.tgz           # fresh DSH_HOME profile → dsh web boots
```

CI = 3 jobs: dsh-plugin-doctor preflight (Ubuntu), test-and-load (Ubuntu), and the Windows fresh-profile `dsh web` boot smoke (the upstream npm CLI lacks the linux-x64 pty prebuild, so boot smoke runs on Windows).

## The 8 plugins: each fills a verified registry gap

| Plugin | Gap filled |
|---|---|
| dsh-dep-audit | dependency supply-chain hygiene (peer resolvability, dist-tag contradiction, staleness, licenses, drift) — live run flags dsh-tools' broken `latest=0.0.1-rc.1` (#2763 class) |
| dsh-llms-forge | llms.txt generator (zero hits in the registry) |
| dsh-cn-boot | China-network bootstrap: probes + mirror/proxy recommendations (zero hits; caught a real HuggingFace timeout locally) |
| dsh-timesheet | wall-clock time tracking from session logs (zero hits; token dashboards were everywhere, time tracking nowhere) |
| dsh-discussions-radar | official Discussions radar (the repo is Discussions-only, but nothing surfaced them to agents) |
| dsh-readme-forge | README generator (zero hits; pairs with llms-forge) |
| dsh-firstrun | first-run health check (toolchain/profile/API key/workspace/registry + next steps) |
| dsh-disk-audit | disk-usage audit (session logs grow to hundreds of MB) |

Method: scan the 916-plugin registry first, skip anything already taken (e.g. dsh-vault is at v1.8.1 with 393 tests — don't compete).

## Real pitfalls (all hit, all fixed)

1. **npm name collision**: `dsh-quickstart` was taken → full rename to `dsh-firstrun` (package, repo, docs, CI, scripts — miss one and CI breaks).
2. **Windows shims**: `spawnSync('pnpm', args)` can't launch pnpm on Windows (it's a .cmd shim) → on win32 build a command string with a shell and a quote helper.
3. **CI grep drift**: after the rename, the smoke grep still used the old id → red CI → fix and rerun green.
4. **0xsline CATALOG.md is CI-generated**: maintainer feedback — hand edits get overwritten; the correct place is README.md + README.zh-CN.md.
5. **dsh-tools `latest` dist-tag is broken** (0.0.1-rc.1 vs declared ^0.1.0-rc.6) — the ecosystem-wide ERESOLVE root cause (#2763); develop against `@next`/rc.6.

## The community loop (publish ≠ done)

1. One Show Your Plugins thread that evolves (#3123) — append updates, don't spam new threads.
2. Answer Q&A with evidence: #55 (cordis-plugin-timer missing on global install) — verified npm metadata + local `require.resolve` before replying.
3. Listing PRs: 0xsline (README edits, not the generated CATALOG) + awesome-dsh-plugin (data/plugins yml + generated README, 1-day gate).
4. Keep your own registry/ecosystem in sync: dsh-subscribe (916 plugins / 29 verified) + dsh-ecosystem.

## Advice for new plugin authors

- Scan for gaps first (registry + official Discussions Ideas).
- Prefer zero runtime dependencies (node: builtins only).
- Read-only by default; any write is an explicit opt-in — that is the trust baseline of this community.
- Never print secret values; show variable names only.
- Bilingual README + llms.txt = discoverability.

## Links

- Repos: github.com/zoahdev/dsh-dep-audit · dsh-llms-forge · dsh-cn-boot · dsh-timesheet · dsh-discussions-radar · dsh-readme-forge · dsh-firstrun · dsh-disk-audit
- Official thread: https://github.com/deepseek-ai/deepseek-harness/discussions/3123
- Ecosystem map: https://github.com/zoahdev/dsh-ecosystem

