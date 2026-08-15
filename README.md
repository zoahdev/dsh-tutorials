# DeepSeek Harness 中文/英文教程集

> 作者：zoahdev · 2026-08-15 · 所有命令实测于 Windows（Node 24 / pnpm 11 / dsh 0.1.0-rc.6）

## 中文

| 篇目 | 内容 |
|---|---|
| [zh-01-start.md](./zh-01-start.md) | 从零上手：安装、启动、Web UI、第一次对话、Windows 踩坑 |
| [zh-02-architecture.md](./zh-02-architecture.md) | 架构解读：一切皆插件、Cordis、工具 DSL、bundle/profile/patch |
| [zh-03-plugin.md](./zh-03-plugin.md) | 手把手写第一个插件（配套 dsh-github-release-radar 源码） |
| [zh-04-contributing.md](./zh-04-contributing.md) | 贡献者路线图：官方认可的方式与 14 天计划 |
| [zh-05-intelligence.md](./zh-05-intelligence.md) | 一分钟尽调任何仓库：dsh-github-intelligence 实战（含真实输出） |
| [zh-06-doctor.md](./zh-06-doctor.md) | 用 dsh-plugin-doctor 给插件做体检（--full 完整链路） |
| [zh-07-awesome.md](./zh-07-awesome.md) | 让插件通过 awesome-dsh-plugin 的 Review：维护者 Checklist |
| [zh-08-peers.md](./zh-08-peers.md) | dsh 插件依赖陷阱：prerelease peer 版本与 ERESOLVE |
| [zh-09-search.md](./zh-09-search.md) | 用 dsh-plugin-search 在 DSH 里找插件（搜→查→体检→装） |
| [zh-10-marketplace.md](./zh-10-marketplace.md) | 在 DSH Marketplace 发布插件（含 allowBuilds 实测坑） |
| [zh-11-prepare-crash.md](./zh-11-prepare-crash.md) | 调试 `undefined.prepare` 崩溃全家桶（#1697/#1763 根因与修复） |
| [zh-12-visibility.md](./zh-12-visibility.md) | 在 dsh 生态被看见：24 小时复盘（可复制 checklist） |

## English

| Post | Content |
|---|---|
| [en-01-start.md](./en-01-start.md) | Getting started: install, run, Web UI, pitfalls |
| [en-02-architecture.md](./en-02-architecture.md) | Architecture summary + glossary |
| [en-03-plugin.md](./en-03-plugin.md) | Build your first plugin (full translation) |
| [en-04-contributing.md](./en-04-contributing.md) | Contributor roadmap (summary) |
| [en-05-intelligence.md](./en-05-intelligence.md) | One-minute repo due diligence with real output |
| [en-06-doctor.md](./en-06-doctor.md) | Health-check your dsh plugin (full chain) |
| [en-07-awesome.md](./en-07-awesome.md) | Passing the awesome-dsh-plugin review checklist |
| [en-08-peers.md](./en-08-peers.md) | Prerelease peers and ERESOLVE troubleshooting |
| [en-09-search.md](./en-09-search.md) | Finding plugins from inside DSH |
| [en-10-marketplace.md](./en-10-marketplace.md) | Publishing to the DSH Marketplace |
| [en-11-prepare-crash.md](./en-11-prepare-crash.md) | Debugging the undefined.prepare crash family |
| [en-12-visibility.md](./en-12-visibility.md) | Getting noticed in the dsh ecosystem (retrospective) |

## 配套资源

- 插件源码（GitHub）：[zoahdev/dsh-github-release-radar](https://github.com/zoahdev/dsh-github-release-radar)
- 旗舰整合（GitHub）：[zoahdev/dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- 插件模板（GitHub）：[zoahdev/dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- 插件医生（GitHub）：[zoahdev/dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 插件搜索（GitHub）：[zoahdev/dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- 在线站点（GitHub Pages）：[zoahdev.github.io/dsh-tutorials](https://zoahdev.github.io/dsh-tutorials/)
- 插件源码（本地副本）：[outputs/deepseek-plugin](../deepseek-plugin/README.md)
- 社区材料包：[outputs/community-kit](../community-kit/README.md)

## 发布建议

- 中文篇适合掘金/知乎/公众号；英文篇适合个人博客 / Dev.to / X。
- 每篇发布时把 `[你的…]` 占位符替换成真实链接。
