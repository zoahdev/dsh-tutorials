# One-Click Plugin Install Inside DSH: the dsh-subscribe In-Harness Market

> By zoahdev · 2026-08-15 · Repo: https://github.com/zoahdev/dsh-subscribe (v0.3.0)

## TL;DR

Install the plugin market into DeepSeek Harness: open `/dsh-subscribe/` in your browser, browse 536 plugins, and install, uninstall, update, or approve build scripts with one click — every button drives the real dsh CLI on your machine.

## 1. Install the market plugin

```sh
dsh plugin --profile web add github:zoahdev/dsh-subscribe#path:/plugin
```

Restart `dsh web`, then open:

```text
http://localhost:<your-dsh-port>/dsh-subscribe/
```

## 2. What the page does

- Browse/search 536 plugins with bilingual descriptions, star or verified sorting
- One-click install with live progress and auto-refreshed installed list
- One-click uninstall / update per installed card
- Build-script approval: when pnpm's `allowBuilds` blocks a git install, the page offers one-click approval (installed packages only)
- Steam-style subscriptions: subscribe → export `subscriptions.json` → one sync command installs everything
- Sanitized log export at `/dsh-subscribe/logs` for bug reports

## 3. Under the hood

The plugin mounts same-origin HTTP routes on the host webServer (see the zh version of this page for the full route table). Security: every mutating route requires a same-origin POST; install specs must come from the curated registry or be explicit `file:`/`link:` specs; spawn targets pass an allowlist regex.

## 4. Evidence (not just "it loads")

```text
fresh DSH profile → install tarball → --dump-config contains dsh-subscribe
→ dsh web boots HTTP 200
→ GET /dsh-subscribe/registry → 536 plugins, 20 verified
→ POST /dsh-subscribe/install (real dsh CLI) → exit 0
→ GET /dsh-subscribe/installed → contains dsh-subscribe
```

## 5. CLI (when you don't want the browser)

```sh
node scripts/dsh-subscribe.mjs list
node scripts/dsh-subscribe.mjs install <id>
node scripts/dsh-subscribe.mjs sync --dry-run
node scripts/dsh-subscribe.mjs sync --profile web
```

## Related

- Web storefront (works without the plugin): https://zoahdev.github.io/dsh-subscribe/
- Agent tools: `market_search` / `market_stats` / `market_install_command`
