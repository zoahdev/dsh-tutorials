# dsh Plugin Dependency Traps: Prerelease Peers and ERESOLVE

> By zoahdev · 2026-08-15 · Behavior verified against dsh 0.1.0-rc.6 (npm and pnpm)

## The semver fact nobody says out loud

`^0.1.0-rc.6` means:

```text
>=0.1.0-rc.6 and <0.2.0
```

It **matches**: `0.1.0-rc.6`, `0.1.0-rc.7`, `0.1.0`, `0.1.1`…
It **does not match**: `0.1.0-rc.3`, `0.0.9`, `0.2.0`.

So `^0.1.0-rc.6` is a floor, not a pin. **If your host has `0.1.0-rc.3` and the plugin requires rc.6, they conflict.**

## npm vs pnpm behavior

| Scenario | npm | pnpm (default) |
|---|---|---|
| Host already has old RC `0.1.0-rc.3` | `ERESOLVE` peer conflict, hard fail | Often a generic warning, installs anyway |
| Host has no dsh-tools | Tries to install the peer | With `autoInstallPeers=false`, does not install — breaks at runtime |
| Host is rc.6 | Fine | Fine |

pnpm's silent path is the worst: the plugin's `apply()` links to the old copy, and the crash happens at the first tool call, not at install time.

## Verified 2026-08-15: npm's `latest` tag is itself a trap

Actual dist-tags for `@deepseek-ai/dsh-tools` on npm:

```text
latest: 0.0.1-rc.1
next:   0.1.0-rc.6
```

A bare `npm install @deepseek-ai/dsh-tools` (or a peer auto-install) gets `0.0.1-rc.1`, not the `0.1.0-rc.6` the ecosystem builds against — and `0.0.x` does not satisfy the `^0.1.0-rc.6` peer range at all. This is a real source of ERESOLVE / peer conflicts.

Install explicitly:

```sh
pnpm add @deepseek-ai/dsh-tools@next   # 0.1.0-rc.6
# or install dsh@0.1.0-rc.6 through the official channel
```

## Three layers of defense (what the template does)

1. **Peer declaration**: only the tested range `^0.1.0-rc.6`, no pretend-wide ranges;
2. **Runtime guard**: `apply()` resolves the actually-linked `@deepseek-ai/dsh-tools` version and throws an actionable error on mismatch (see [dsh-plugin-template/src/version.ts](https://github.com/zoahdev/dsh-plugin-template));
3. **Install verification**: CI installs the tarball into a fresh host and really invokes the tool (the template's integration test).

## User side: ERESOLVE / peer conflict

```sh
# 1. Find the actual version in your host
pnpm why @deepseek-ai/dsh-tools
# or
npm ls @deepseek-ai/dsh-tools

# 2. Upgrade the host to the version the plugin was verified against (0.1.0-rc.6), then install
dsh upgrade
dsh plugin --profile web add <your-plugin>

# 3. Confirm what is actually composed
dsh --profile web --dump-config
```

Do **not** reach for `--legacy-peer-deps`: it hides the conflict and the plugin will likely explode at runtime anyway.

## Plugin author side: before publishing

```sh
node /path/to/dsh-plugin-doctor/lib/bin.js --full .
```

The `install` check really runs `dsh plugin add` + `--dump-config`; combine that with a real tool invocation in a fresh host and you have actually verified peer compatibility.

## Resources

- Template (with guard source): [dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- Health checks: [dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- Official publishing guide: [publish.md](https://github.com/deepseek-ai/deepseek-harness/blob/HEAD/docs/user/develop/basic/publish.md)
