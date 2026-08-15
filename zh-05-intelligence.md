# 一分钟尽调任何 GitHub 仓库：dsh-github-intelligence 实战

> 作者：zoahdev · 2026-08-15 · 本文输出全部来自真实 GitHub API 数据

## 目录

- [装一个就能用](#装一个就能用)
- [三个最常用的提问](#三个最常用的提问)
- [真实输出长什么样](#真实输出长什么样)
- [7 个工具一览](#7-个工具一览)
- [为什么它省配额](#为什么它省配额)
- [踩坑：issues 端点会混入 PR](#踩坑issues-端点会混入-pr)
- [下一步](#下一步)

## 装一个就能用

```sh
dsh plugin --profile web add github:zoahdev/dsh-github-intelligence
```

重启 `dsh web`，然后直接问。无需 API Key（匿名 60 次/小时，插件内置 60 秒缓存帮你省着用）。

## 三个最常用的提问

1. "Give me a full report on `deepseek-ai/deepseek-harness`."
2. "What open issues does `ollama/ollama` have right now?"
3. "Compare the top 5 TypeScript agent frameworks by stars."

## 真实输出长什么样

这是 `github_repo_report` 在 2026-08-15 对 `deepseek-ai/deepseek-harness` 的真实输出：

```text
# deepseek-ai/deepseek-harness
deepseek-ai/deepseek-harness — DeepSeek Harness: Everything is a Plugin.
Stars: 98472 · Forks: 9214 · Open issues: 0 · Default branch: master
Language: TypeScript · License: MIT · Pushed: 2026-08-13
https://github.com/deepseek-ai/deepseek-harness

Latest release: none
Open issues: 0

Recent commits:
- 47f9438 Merge pull request #2519 from deepseek-harness/feat/npm-public (imccyu)
- abe560f release(dsh): 0.1.0-rc.5 (imccyu)
- 8c1e8d9 build(release): publish the dsh family publicly (imccyu)
- f26a6f6 Merge pull request #2520 from deepseek-harness/docs/paper (Tianyi Cui)
- 124aa5f Merge pull request #2521 from deepseek-harness/release/dsh-0.1.0-rc.3 (imccyu)

Top contributors: 1. tianyicui (5235) · 2. LegGasai (1361) · 3. imccyu (1168) · 4. Chinesezjc (587) · 5. turtle1999 (585)
```

一次提问 = 仓库全景 + 最新 Release + open issues + 最近提交 + 头部贡献者。这就是"尽调"。

## 7 个工具一览

| 工具 | 作用 |
|---|---|
| `github_repo` | 仓库全景（含话题、默认分支、归档状态） |
| `github_releases` | 最近 Release + 正文预览 |
| `github_issues` | 按状态查 Issue |
| `github_pulls` | 按状态查 PR（含合并状态） |
| `github_contributors` | 头部贡献者排名 |
| `github_search` | 按星标/更新时间搜索 |
| `github_repo_report` | 上面五类数据一键合成 |

## 为什么它省配额

匿名 GitHub API 只有 60 次/小时。插件所有接口都有 60 秒 TTL 缓存：同一仓库连续问 3 遍报告，实际只发 4 个请求（第一次），后面全部命中缓存。深度报告内部也是并行 + 复用缓存，不是简单叠加。

## 踩坑：issues 端点会混入 PR

GitHub 的 `/issues` 端点默认把 PR 也返回。插件在 `listIssues` 里按 `pull_request` 字段自动过滤——这也是"整合"和"调 API"的区别：表面一样，数据口径干净。

## 下一步

- 源码与验证记录：[zoahdev/dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- 更多教程：[教程索引](./README.md)
