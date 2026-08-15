# 用 dsh-plugin-search 在 DSH 里找插件

> 作者：zoahdev · 2026-08-15 · 命令实测于 Windows（Node 24 / pnpm 11 / dsh 0.1.0-rc.6）

## 为什么需要它

写插件前先搜一遍，是最省时间的习惯。dsh 生态已经有不少插件，但分散在 npm 和 awesome 清单里——[dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search) 让 agent 在 DSH 里直接搜，不用切出去翻网页。

## 安装

```sh
dsh plugin --profile web add https://github.com/zoahdev/dsh-plugin-search/releases/download/v1.0.0/dsh-plugin-search-1.0.0.tgz
```

## 三个工具

| 工具 | 作用 |
|---|---|
| `dsh_search_plugins` | 同时搜 npm registry + awesome-dsh-plugin 精选清单，npm 优先、去重、带版本与来源 |
| `dsh_plugin_lookup` | 按精确包名查 npm 元数据（最新版本/描述/主页/仓库/作者） |
| `dsh_awesome_top` | 浏览当前精选清单前 N 条 |

## 在 DSH 里对 agent 说

> 帮我找一个能监控 GitHub release 的 dsh 插件，然后查一下它的详情。

agent 会先调 `dsh_search_plugins("github release")`，再对最相关的结果调 `dsh_plugin_lookup`，把版本、仓库、描述一起给你。

## 完整工作流：搜 → 查 → 体检 → 装

```text
1. dsh_search_plugins("github release")   # 找候选
2. dsh_plugin_lookup("dsh-github-release-radar")  # 看详情
3. dsh-plugin-doctor 的 plugin_check       # 发布前/装前体检（manifest/patch/build/install）
4. dsh plugin --profile web add <tarball>  # 安装
```

这套组合把"找插件"到"放心安装"串成一条链。

## 注意事项

- `dsh_plugin_lookup` 返回的是 npm 的 `latest` 标签版本。对 `@deepseek-ai/dsh-tools` 这类包，`latest` 可能是旧 RC（实测 `0.0.1-rc.1`），真正在用的是 `next` 标签（`0.1.0-rc.6`）——判断插件兼容性时看具体版本，别只看 latest（详见 [prerelease peer 排障](./zh-08-peers.html)）。
- 搜索关键词用任务描述比用包名更容易命中（例如 "github release" 而不是 "radar"）。

## 相关资源

- 仓库：[zoahdev/dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- 官方讨论 #1715：[链接](https://github.com/deepseek-ai/deepseek-harness/discussions/1715)
