# Health-Check Your dsh Plugin with dsh-plugin-doctor

> By zoahdev · 2026-08-15 · All commands tested on Windows (Node 24, pnpm 11)

## Table of Contents

- [Why you need a health check](#why-you-need-a-health-check)
- [Quick start](#quick-start)
- [Checks](#checks)
- [Full check: --full](#full-check---full)
- [CI integration](#ci-integration)
- [FAQ](#faq)

## Why you need a health check

"The plugin loads" is not "the plugin works". Two real failure classes:

1. **Broken manifest structure** — a missing `dsh.bundle`, `cordis.patch.yml`, or `prepare` script explodes on someone else's machine;
2. **Silently mis-linked peers** — pnpm's default config can link an old `@deepseek-ai/dsh-tools` RC into your plugin (verified: `0.1.0-rc.3` linked against `^0.1.0-rc.6` with only a generic warning).

[dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor) turns both into "run it locally and know immediately".

## Quick start

```sh
git clone https://github.com/zoahdev/dsh-plugin-doctor.git
cd dsh-plugin-doctor
pnpm install
pnpm build

node lib/bin.js /path/to/my-plugin
```

Example output:

```text
[PASS] manifest: my-plugin@1.0.0 bundle manifest ok
[PASS] patch: 1 plugin row(s): my-hello
[WARN] entry: entry lib/index.js not built yet (run pnpm install && pnpm run build)
[PASS] files: ships: lib, cordis.patch.yml
✅ ALL CHECKS PASSED
```

Exit code is `0` when everything passes, `1` otherwise.

## Checks

| Check | Verifies | Default |
|---|---|---|
| `manifest` | `package.json` + `dsh.bundle` + `dsh.bundle.patch` + `prepare` + `main` | ✅ |
| `patch` | `cordis.patch.yml` parses and has an `insert` row with an `id` | ✅ |
| `entry` | the `main` target exists | ✅ |
| `files` | a `files` allowlist is declared | ✅ |
| `build` | `pnpm run build` actually runs | `--build` |
| `pack`+`install` | pack → install into a fresh DSH profile → plugin id in `--dump-config` | `--full` |

## Full check: --full

```sh
node lib/bin.js --full /path/to/my-plugin
```

This is the closest local simulation of "installed on someone else's machine":

```text
[PASS] manifest: dsh-plugin-template@0.1.0 bundle manifest ok
[PASS] patch: 1 plugin row(s): template-hello
[PASS] entry: entry lib/index.js exists
[PASS] files: ships: lib, cordis.patch.yml, README.md, LICENSE
[PASS] pack: packed dsh-plugin-template-0.1.0.tgz
[PASS] install: plugin id(s) present in composed config: template-hello
✅ ALL CHECKS PASSED
```

## Plugin shell (v1.1.0): let the agent run the check

Since v1.1.0, dsh-plugin-doctor also ships a plugin shell (`dsh.bundle` + `cordis.patch.yml`), so it can be installed straight into DSH:

```sh
dsh plugin --profile web add dsh-plugin-doctor
# or from a local tarball:
dsh plugin --profile web add ./dsh-plugin-doctor-1.1.0.tgz
```

Then tell the agent inside DSH:

> Check whether this plugin is ready to publish — run the build first, then do a full verification.

The agent calls the `plugin_check` tool (`dir` plus optional `build`/`full`), returning PASS/WARN/FAIL per check and an overall `ok` — no need to leave DSH.

## CI integration

```sh
node lib/bin.js --json ./my-plugin
```

`--json` gives a machine-readable report plus the exit code, ready for GitHub Actions:

```yaml
- run: node /path/to/dsh-plugin-doctor/lib/bin.js --build --json ./my-plugin
```

## FAQ

**Q: `entry` warns?** The plugin is not built yet: `pnpm install && pnpm run build`.

**Q: `manifest` fails?** Compare with the [official publishing guide](https://github.com/deepseek-ai/deepseek-harness/blob/HEAD/docs/user/develop/basic/publish.md) and add `dsh.bundle` + `prepare`.

**Q: `install` fails?** The plugin id is not in the composed config — check the `id` in `cordis.patch.yml`, or upgrade the host to 0.1.0-rc.6+ and retry.

## Related resources

- Repo: [zoahdev/dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- Companion template: [zoahdev/dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template) (its CI already includes a real tool-invocation smoke)
- Proposal: [RFC #1629](https://github.com/deepseek-ai/deepseek-harness/discussions/1629)
