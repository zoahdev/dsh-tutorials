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
| [zh-13-market.md](./zh-13-market.md) | 在 DSH 里一键装插件：dsh-subscribe in-harness 市场实战（v0.3） |
| [zh-14-doctor-triage.md](./zh-14-doctor-triage.md) | dsh-plugin-doctor 排障手册：profile-shadow / manifest-bom / dsh-doctor/v1 契约 |
| [zh-15-retrieval-efficiency.md](./zh-15-retrieval-efficiency.md) | 高效代码库检索：AGENTS.md 工具优先约定（#1864 效率反馈） |
| [zh-16-self-evolution.md](./zh-16-self-evolution.md) | 让 agent 自我进化：dsh-rule-evolve 实战（验证驱动闭环） |
| [zh-17-growth-report.md](./zh-17-growth-report.md) | 让 agent 的成长看得见：Agent Growth Report 实战 |

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
| [en-13-market.md](./en-13-market.md) | One-click plugin install inside DSH (in-harness market) |
| [en-14-doctor-triage.md](./en-14-doctor-triage.md) | doctor triage: profile-shadow / manifest-bom / v1 contract |
| [en-15-retrieval-efficiency.md](./en-15-retrieval-efficiency.md) | Efficient retrieval: a tools-first AGENTS.md contract |
| [en-16-self-evolution.md](./en-16-self-evolution.md) | Let agents evolve: dsh-rule-evolve in practice |
| [en-17-growth-report.md](./en-17-growth-report.md) | Make agent growth visible: the growth report |
| [zh-18-lan-deploy.md](./zh-18-lan-deploy.md) | 内网部署 dsh web：非安全上下文崩溃的根因与修法 |
| [en-18-lan-deploy.md](./en-18-lan-deploy.md) | LAN deployment: insecure-context crash root cause + fix |
| [zh-19-compaction-cache.md](./zh-19-compaction-cache.md) | compaction 缓存击穿：为什么每次自动压缩都多付钱 |
| [en-19-compaction-cache.md](./en-19-compaction-cache.md) | Compaction cache misses: why auto-compaction re-bills |
| [zh-20-pet-evolve.md](./zh-20-pet-evolve.md) | dsh-pet-evolve：把 agent 成长变成一只宠物 |
| [en-20-pet-evolve.md](./en-20-pet-evolve.md) | dsh-pet-evolve: turn agent growth into a pet |
| [zh-21-evolution-badge.md](./zh-21-evolution-badge.md) | 进化徽章：把规则库变成 README 上的 SVG |
| [en-21-evolution-badge.md](./en-21-evolution-badge.md) | The evolution badge: rule library as a README SVG |
| [zh-22-session-shelf.md](./zh-22-session-shelf.md) | dsh-shelf：给你的 dsh 会话一个书架 |
| [en-22-session-shelf.md](./en-22-session-shelf.md) | dsh-shelf: give your dsh sessions a shelf |

## 配套资源

- 插件源码（GitHub）：[zoahdev/dsh-github-release-radar](https://github.com/zoahdev/dsh-github-release-radar)
- 旗舰整合（GitHub）：[zoahdev/dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- 插件模板（GitHub）：[zoahdev/dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- 插件医生（GitHub）：[zoahdev/dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 插件搜索（GitHub）：[zoahdev/dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- 插件市场（GitHub）：[zoahdev/dsh-subscribe](https://github.com/zoahdev/dsh-subscribe)
- 在线站点（GitHub Pages）：[zoahdev.github.io/dsh-tutorials](https://zoahdev.github.io/dsh-tutorials/)
- 插件源码（本地副本）：[outputs/deepseek-plugin](../deepseek-plugin/README.md)
- 社区材料包：[outputs/community-kit](../community-kit/README.md)

## 发布建议

- 中文篇适合掘金/知乎/公众号；英文篇适合个人博客 / Dev.to / X。
- 每篇发布时把 `[你的…]` 占位符替换成真实链接。
