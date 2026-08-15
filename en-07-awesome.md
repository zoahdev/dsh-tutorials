# Passing the awesome-dsh-plugin Review: The Maintainer Checklist

> By zoahdev · 2026-08-15 · Based on maintainer fkysly's actual review of PR #352

## The boundary

[awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) only lists plugins installable with a single `dsh plugin add` (boundary discussion: [PR #248](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/248)).

A pure CLI tool is out of scope unless published to npm for `npx`. The maintainer's words: "a health checker an agent can call is far more valuable to the ecosystem than a CLI". Either make it a plugin, or don't submit it here.

## Checklist

| # | Item | Failure mode | How to verify yourself |
|---|---|---|---|
| 1 | `dsh.bundle` in package.json | Only `bin`, no `dsh` field | `node -e "console.log(require('./package.json').dsh)"` |
| 2 | `dsh.bundle.patch` points at an existing `cordis.patch.yml` | Plugin id missing from dump-config | `dsh --profile web --dump-config` |
| 3 | `cordis.patch.yml` valid, with an `insert` row carrying an `id` | YAML error / empty list | doctor's `patch` check |
| 4 | `files` allowlist includes build output | Someone installs and gets only `src/` | `pnpm pack --dry-run` |
| 5 | `prepare` script | Git installs never build | `npm pkg get scripts.prepare` |
| 6 | At least one model tool (agent-callable) | Installs but the agent cannot use it | `ctx.tools.register` in `apply()` |
| 7 | CI proves *real callability* | Only proves loadability | Install tarball → register → execute handler → assert |
| 8 | README covers install + usage | Nobody knows what to do next | Follow your own README |
| 9 | Real install verification | Never installed it yourself | `dsh plugin --profile web add <tarball>` |

## One command covers ~80%

```sh
git clone https://github.com/zoahdev/dsh-plugin-doctor.git
cd dsh-plugin-doctor && pnpm install && pnpm build
node lib/bin.js --build --full /path/to/your-plugin
```

Covers: manifest / patch / entry / files / build / pack / fresh-profile install / dump-config.

## The three most common rejections (maintainer's own words)

1. **No `dsh.bundle`** — only `bin`, no plugin shell;
2. **`files` lists only `src/`** — `lib/` not shipped, no `prepare`;
3. **Pure CLI submitted as a plugin** — it cannot be installed, and it is outside the list boundary.

## What to do after a rejection

A closed PR is not the end. In [PR #352](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/352):

1. The maintainer offered two paths (pure CLI → npm `npx`; or add a plugin shell);
2. We took the plugin-shell path: `cordis.patch.yml` + `dsh.bundle` + a `plugin_check` model tool, keeping the `bin`;
3. We proved every item with real CI (packaged install + real invocation + self-check), then asked for a re-review.

Treat the review as an acceptance spec, not a debate.

## Resources

- List: [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin)
- Health checks: [dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- Template: [dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
