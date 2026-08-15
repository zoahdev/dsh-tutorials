# 在 dsh 生态里被看见：贡献者路线图

> 作者：zoahdev · 2026-08-15 · 基于官方 CONTRIBUTING.md 与社区现状

## 目录

- [官方现状：PR 暂不收](#官方现状pr-暂不收)
- [官方认可的四种贡献方式](#官方认可的四种贡献方式)
- [社区渠道](#社区渠道)
- [新手路线图（14 天）](#新手路线图14-天)
- [注意事项](#注意事项)

## 官方现状：PR 暂不收

DeepSeek Harness 目前处于开发者预览，官方 [CONTRIBUTING.md](https://github.com/deepseek-ai/deepseek-harness/blob/HEAD/CONTRIBUTING.md) 明确写道：

> We are sorry that we cannot accept external pull requests at the moment.

但同一份文档列出了大量其他参与方式——**贡献代码远远不是唯一的方式**。官方团队很小，会认真看 GitHub Discussions 并按反馈分配资源。

## 官方认可的四种贡献方式

### 1. Discussions：反馈与 bug 报告

- 在 https://github.com/deepseek-ai/deepseek-harness/discussions 提交 bug/反馈；
- 给想推进的讨论**点赞（upvote）**——这是官方的资源分配信号；
- 报告必须可复现：环境、版本、复现步骤、期望/实际、影响。

### 2. 插件生态（重点）

- 做一个让"你自己也兴奋"的插件；
- 仓库加 **`dsh-plugin`** 话题：https://github.com/topics/dsh-plugin；
- 官方说得很直白：**官方包不比社区包重要**，这个仓库只是"想法、官方展示和灵感来源"。

### 3. 教程与指南

- 写博客、how-to 指南；
- 中文社区内容极度稀缺，双语内容价值更高；
- 所有命令必须实测，别编造输出——技术社区一次造假就毁掉信任。

### 4. 答疑

- 回答其他成员的问题；
- 这是建立"这人靠谱"印象的最快方式。

## 社区渠道

| 渠道 | 入口 | 适合 |
|---|---|---|
| GitHub Discussions | [链接](https://github.com/deepseek-ai/deepseek-harness/discussions) | 官方看得见的反馈 |
| Discord | https://discord.gg/Ycq5dCaS4 | 实时答疑、英文社区 |
| 企微群 | 官方 README 二维码（扫码加小助手+填问卷） | 中文深度交流 |
| dsh-plugin 话题 | https://github.com/topics/dsh-plugin | 插件被发现 |

## 新手路线图（14 天）

1. **第 1-2 天**：`npx @deepseek-ai/dsh web` 跑起来；读完 AGENTS.md 和开发文档；提交入群申请。
2. **第 3-4 天**：做一个小而真的插件（参考[手把手教程](./zh-03-plugin.md)）；双语 README；打 `dsh-plugin` 话题。
3. **第 5-6 天**：发布上手/插件教程（掘金、知乎、公众号任选）。
4. **第 7-8 天**：把你实际遇到的 bug/文档缺口发到 Discussions（先搜索防重复）。
5. **第 9-14 天**：持续答疑 + 根据反馈迭代插件 v0.2；写复盘文；在 Discord/企微群分享链接。

## 注意事项

- 不刷屏、不重复提问、不标题党；
- 所有论断要有源码或实测依据；
- 插件不硬编码可配置值（超时、条数、开关都进 Config）；
- 提交前跑针对性检查（官方 AGENTS.md 有完整约定）；
- 记住早期项目迭代极快，**会有破坏性变更**，README 里注明你测试的版本。
