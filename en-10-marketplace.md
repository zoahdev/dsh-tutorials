# Publishing Your Plugin to the DSH Marketplace

> By zoahdev · 2026-08-15 · Based on a real submission (ydhrdh/dsh-marketplace PR #1) and real installs

## What it is

[dsh-marketplace](https://github.com/ydhrdh/dsh-marketplace) (discussion [#1752](https://github.com/deepseek-ai/deepseek-harness/discussions/1752)) is an open-source plugin marketplace: one JSON manifest per plugin, PR-based review, git history as the audit log, zero-cost static hosting.

## Requirements

- The plugin is publicly installable: npm (`npm`), git (`github:owner/repo`), or local (`local`).
- The manifest passes `npm run validate`.
- `install.command` uses the `{profile}` placeholder.
- The repo carries the `dsh-plugin` topic.

## Minimal manifest

Required: `id` (matches the directory), `name`, `version`, `description`, `author{name,url}`, `license`, `category`, `install{target,spec,command}`, `repository`. Set `verified: false` on first submission; maintainers flip it after review.

```json
{
  "id": "dsh-plugin-doctor",
  "name": "Plugin Doctor",
  "version": "1.3.0",
  "category": "tools",
  "install": {
    "target": "git",
    "spec": "github:zoahdev/dsh-plugin-doctor",
    "command": "dsh plugin --profile {profile} add github:zoahdev/dsh-plugin-doctor"
  },
  "repository": "https://github.com/zoahdev/dsh-plugin-doctor"
}
```

## Flow

```sh
git clone https://github.com/ydhrdh/dsh-marketplace
mkdir -p registry/plugins/<your-plugin-id>
npm install
npm run validate
npm run build
# open a PR; CI must be green
```

## Real trap: git installs are blocked by allowBuilds

Installing a git-hosted plugin triggers pnpm 11's build-script gate:

```text
[ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED] ... not in the "allowBuilds" allowlist
```

Add the exact key the CLI prints to the profile's `pnpm-workspace.yaml`, then re-run `dsh plugin add`. It succeeds and `--dump-config` shows the plugin row.

**Tip**: include your real install output (both the failure and the success) in the PR description — it makes maintainer review much faster. That's exactly how our PR #1 is written.

## References

- Our submission PR: https://github.com/ydhrdh/dsh-marketplace/pull/1 (5 plugins)
- Marketplace repo: https://github.com/ydhrdh/dsh-marketplace
- Live site: https://ydhrdh.github.io/dsh-marketplace/
