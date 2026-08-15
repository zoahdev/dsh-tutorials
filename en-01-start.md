# Getting Started with DeepSeek Harness: Install, Run, and Chat

> By zoahdev · 2026-08-15 · Every command below was tested on Windows (Node 24, pnpm 11)

## Table of Contents

- [What is DeepSeek Harness?](#what-is-deepseek-harness)
- [Requirements](#requirements)
- [Option 1: One-line npm start (recommended)](#option-1-one-line-npm-start-recommended)
- [Option 2: Run from source](#option-2-run-from-source)
- [Your first chat: configure a model](#your-first-chat-configure-a-model)
- [Web UI overview](#web-ui-overview)
- [Common pitfalls (all tested)](#common-pitfalls-all-tested)
- [FAQ](#faq)

## What is DeepSeek Harness?

DeepSeek Harness (`dsh`) is the first agent product open-sourced by DeepSeek (2026-08-13, MIT). It is **not a model** — it is the infrastructure that lets a model use tools, run tasks, and collaborate with developers. Its core design is "everything is a plugin", powered by Cordis.

> Repo: https://github.com/deepseek-ai/deepseek-harness
> Status: developer preview v0.1, iterating fast, **breaking changes expected** (official warning).

## Requirements

- Node.js `^22.19.0 || >=24.0.0` (official)
- pnpm (10+; the repo uses 11.x)
- A DeepSeek API key (for chat and web search)

Verify:

```sh
node -v
pnpm -v
```

## Option 1: One-line npm start (recommended)

```sh
npx @deepseek-ai/dsh web
```

Tested output (dsh 0.1.0-rc.6):

```text
dsh web: http://127.0.0.1:3080
```

Open http://127.0.0.1:3080. If port 3080 is busy:

```sh
npx @deepseek-ai/dsh web --port 4099
```

## Option 2: Run from source

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

`pnpm install` takes a few minutes (large pnpm workspace). When running from source, the CLI boots `apps/cli/src/bin.ts` through tsx.

> Read `AGENTS.md` at the repo root before contributing — it encodes important conventions (registrations are effects, run targeted checks before pushing, etc.).

## Your first chat: configure a model

1. Open **Providers / Models** in the UI.
2. The default model is `deepseek-v4-flash` (provider `deepseek-official`).
3. Enter your DeepSeek API key, or set it before starting:

```sh
# PowerShell
$env:DEEPSEEK_API_KEY = "sk-your-key"
```

4. Start a session and ask something like: "Introduce yourself and list the tools you can use."

Without a key the UI opens but chat fails with an auth error — expected behavior, not a bug.

## Web UI overview

- **Sessions**: state (todos, plans, file changes) is isolated per session.
- **Tool cards**: tools declare pending/completed cards (`generic`, `terminal`, `diff`, `search`, `web`) via `presentCall`/`presentResult`.
- **Models**: pick providers/models, including OpenAI-compatible endpoints.
- **Timezone**: the browser attaches its IANA zone to each request, so "today/tomorrow" is interpreted in your zone.

## Common pitfalls (all tested)

### 1. Port 3080 already in use

```text
listen EADDRINUSE: address already in use 127.0.0.1:3080
```

Fix: `--port 0` (OS picks a free port) or `--port 4099`.

### 2. `node`/`pnpm` not found

Install Node, then **open a new terminal**. Enable pnpm with `corepack enable` or `npm install -g pnpm`.

### 3. Version mismatch on npm

Community plugins must target the current release train `0.1.0-rc.6`. The npm `latest` tag of `@deepseek-ai/dsh-tools` still points at `0.0.1-rc.1`, whose peer `@deepseek-ai/dsh-user-approval@0.0.1-rc.1` depends on the **unpublished** `@deepseek-ai/dsh-type-meta` — a standalone install 404s. Pin `^0.1.0-rc.6`.

### 4. Git plugin installs need build permission

`dsh plugin add github:owner/repo` runs the package's `prepare` script; pnpm refuses by default. `dsh` prints the exact `allowBuilds:` line to add to the profile's `pnpm-workspace.yaml`. Only allow it for source you trust.

## FAQ

**Q: Does dsh need a GPU?** No. It calls the DeepSeek API (or any OpenAI-compatible provider).

**Q: Does the official repo accept PRs?** Not yet — see CONTRIBUTING.md. Discussions, plugins, tutorials, and answering questions are the official contribution paths.

**Q: `web` vs `--profile web`?** `dsh web` is an alias of `dsh --profile web`. Use `dsh plugin --profile web add <pkg>` to install plugins into the web profile.

Next: [Understanding dsh Architecture: Everything is a Plugin and Cordis](./en-02-architecture.md)
