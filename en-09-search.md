# Finding dsh Plugins from Inside DSH with dsh-plugin-search

> By zoahdev · 2026-08-15 · Commands tested on Windows (Node 24, pnpm 11, dsh 0.1.0-rc.6)

## Why

Search before you build. The dsh ecosystem already has plugins scattered across npm and the awesome list; [dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search) lets the agent search directly inside DSH instead of tabbing out to a browser.

## Install

```sh
dsh plugin --profile web add https://github.com/zoahdev/dsh-plugin-search/releases/download/v1.0.0/dsh-plugin-search-1.0.0.tgz
```

## The three tools

| Tool | What it does |
|---|---|
| `dsh_search_plugins` | Searches npm registry + the awesome-dsh-plugin list, npm-first, deduplicated, with version + source labels |
| `dsh_plugin_lookup` | Exact npm metadata lookup (latest version, description, homepage, repository, author) |
| `dsh_awesome_top` | Browses the current curated list |

## Tell the agent

> Find me a dsh plugin that tracks GitHub releases, then look up its details.

The agent calls `dsh_search_plugins("github release")`, then `dsh_plugin_lookup` on the best hit.

## Full workflow: search → inspect → health-check → install

```text
1. dsh_search_plugins("github release")      # find candidates
2. dsh_plugin_lookup("dsh-github-release-radar")  # inspect
3. dsh-plugin-doctor plugin_check             # pre-install health check
4. dsh plugin --profile web add <tarball>     # install
```

## Caveats

- `dsh_plugin_lookup` reports npm's `latest` dist-tag. For packages like `@deepseek-ai/dsh-tools`, `latest` can be an old RC (`0.0.1-rc.1` was measured) while the ecosystem runs `next` (`0.1.0-rc.6`) — check the actual version, not the tag (see [prerelease peers](./en-08-peers.html)).
- Task-description queries hit better than package names ("github release" beats "radar").

## Resources

- Repo: [zoahdev/dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- Discussion #1715: [link](https://github.com/deepseek-ai/deepseek-harness/discussions/1715)
