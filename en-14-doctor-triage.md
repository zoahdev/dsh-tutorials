# dsh-plugin-doctor Triage: profile-shadow / manifest-bom / the dsh-doctor/v1 Contract

> By zoahdev · 2026-08-15 · Repo: https://github.com/zoahdev/dsh-plugin-doctor (v1.6.0)

## When DSH misbehaves, run one command

```sh
npx dsh-plugin-doctor --profile ~/.dsh/profiles/web --json
```

Exit codes: `0` all pass / `1` warnings / `2` FAIL.

## Check 1: profile-shadow (#1697 dual-instance)

Symptom: after installing any plugin that depends on `@deepseek-ai/dsh-tools`, every tool call crashes with `Cannot read properties of undefined (reading 'prepare')`, and the session gets stuck.

Root cause: with `nodeLinker: hoisted`, the plugin's `dsh-tools` copy is hoisted to the profile's top-level `node_modules`; the host and the copy use different `TOOL_RUNTIME_SCHEDULER` symbols.

Diagnostic: `profile-shadow: fail` naming the shadowed packages. Fix direction: `Symbol.for` + protocol-version guard (cherry-pick-ready branch exists); temporary workaround: reinstall the profile so the host copy resolves first.

## Check 2: manifest-bom (#1842 boot crash)

Symptom: `dsh web` crashes at boot with `Unexpected token` and no BOM hint.

Root cause: the profile's `package.json` starts with a UTF-8 BOM; `readProfileManifest` calls `JSON.parse(readFileSync(..., 'utf8'))` directly.

Diagnostic: `manifest-bom: fail`. Fix: re-save as UTF-8 without BOM (or drop the first three bytes). Upstream one-line patch is staged on `fix/profile-manifest-bom-strip`.

## Output contract: dsh-doctor/v1

`--json` always emits the shared envelope (schema, exitCode, summary, lowercase check statuses), so CI, marketplaces, and support flows can consume either doctor implementation interchangeably. Cross-implementation acceptance harness:

```sh
node scripts/doctor-contract-check.mjs --impl1 "node lib/bin.js"
```

It runs clean / BOM / shadow fixtures and asserts exit codes and envelope shape.

## Check 3: shell-launcher (#1923 approval-bypass channel)

Symptom: a plugin can delegate execution to a user-privileged shell
(child_process + explorer/start/open/powershell/cmd), bypassing approval and
workspace-write limits.

Diagnostic: `shell-launcher: warn` naming the file and pattern. Approval is
consent UX, not an OS boundary — an OS-level sandbox (restricted token /
container) is the real enforcement. Confirm such channels are allowlisted or
sandboxed before shipping.
