# Debugging dsh Plugin Crashes: the `undefined.prepare` Family

> By zoahdev · 2026-08-15 · Based on the verified root cause and fix in #1697 / #1763

## Symptom

```text
Cannot read properties of undefined (reading 'prepare')
```

Confusing variants: every tool call crashes after installing a third-party plugin; or only some tools (web_search/fetch) fail intermittently; retries sometimes work. dsh 0.1.0-rc.6, Windows/Linux.

## Root cause (one sentence)

`@deepseek-ai/dsh-tools` keys its internal scheduler with `Symbol('@deepseek-ai/dsh-tools.scheduler')` — and a unique symbol's identity is per module instance. When a second physical copy exists in the profile (pnpm hoists the plugin's transitive SDK to the profile top level), host and copy hold different symbols. `dsh-agent-loop` reads with the host key, the ToolRuntime was stored under the copy's key → `undefined.prepare`.

The intermittency: only calls routed through the *other* instance's scheduler slot crash.

## Diagnose in three steps

```sh
# 1. One-command profile check (real-directory @deepseek-ai/* copies)
node /path/to/dsh-plugin-doctor/lib/bin.js --profile ~/.dsh/profiles/web

# 2. Manual confirmation
ls ~/.dsh/profiles/web/node_modules/@deepseek-ai

# 3. Actual resolved version
pnpm why @deepseek-ai/dsh-tools
```

## Immediate fix: `link:` dependency

Pin the host copy in the profile's `package.json` so the profile and host share one physical dsh-tools (same realpath = same module instance = same symbol):

```jsonc
{
  "dependencies": {
    "@deepseek-ai/dsh-tools": "link:C:/Users/<user>/AppData/Roaming/npm/node_modules/@deepseek-ai/dsh/node_modules/@deepseek-ai/dsh-tools"
  }
}
```

Then run `pnpm install` in the profile and retry.

## Upstream fix (staged, waiting for the PR channel to open)

https://github.com/zoahdev/deepseek-harness/tree/fix/tool-runtime-scheduler-symbol-for

1. `Symbol.for(...)` — all copies in one process share the key;
2. `TOOL_RUNTIME_SCHEDULER_PROTOCOL_VERSION` + `assertSchedulerProtocol()` — version-skewed copies still fail loudly instead of silently running an incompatible protocol.

Verified against the published rc.6 package: two physical copies produce different symbols before the change, identical after.

## Don't confuse it with allowBuilds

`ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED` at install time is pnpm 11 blocking git deps from running `prepare` — a different problem. Add the key the dsh CLI prints to `allowBuilds` in the profile's `pnpm-workspace.yaml` and re-run the install.

## Prevention

- Before publishing: `dsh-plugin-doctor --full .` (real install into a fresh profile).
- The template ships a runtime guard that throws an actionable error on dsh-tools version mismatch ([dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)).
- When you see the crash, run `--profile` first — don't rush to reinstall.

## Threads

- #1697 (root cause + fix): https://github.com/deepseek-ai/deepseek-harness/discussions/1697
- #1763 (same crash, new report): https://github.com/deepseek-ai/deepseek-harness/discussions/1763
