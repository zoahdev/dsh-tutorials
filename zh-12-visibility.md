# 在 dsh 生态被看见：24 小时复盘（5 个仓库 / 195+ 工具 / 23 页教程）

> 作者：zoahdev · 2026-08-15 · 全部内容可对照本系列教程与 GitHub 上的真实提交验证

## 先说结论

开源早期生态里，"出名"没有捷径，但有**可复制的路径**：找官方讨论里被点名缺的东西 → 做出来并给出真实运行证据 → 回应 review 用证据而不是嘴 → 和同赛道工具互相挂链接 → 把知识沉淀成教程。一天下来我们的产出：5 个仓库、195+ 工具、23 页教程、10+ 条官方讨论回复、2 个在途收录 PR。

诚实声明：star 目前还是 0。渠道全部铺好，等外部审核和流量。这篇文章讲的是"怎么让被看见的概率最大"，不是"刷量"。

## 五件事，按顺序

### 1. 做官方讨论里"被点名缺"的东西

三个需求都是官方讨论直接点名的：

- RFC #1629 要 `dsh plugin check` → 我们做了 dsh-plugin-doctor；
- #1715 问"缺 dsh-plugin-search" → 我们做了 dsh-plugin-search；
- #1719 提议 `dsh doctor` → 我们的 `--env` 把它落地。

被点名意味着"有人已经证明了这个需求"，你只需要证明"你做了且能用"。

### 2. 每个功能都跑真实 API 冒烟

我们的验证哲学：**能加载 ≠ 真能用**。每个新工具都对着真实 API 跑一遍，于是拦下了一串"文档写着公开、实际会挂"的坑：

- Bitbucket Cloud：匿名访问 404/410 → 不收录；
- Maven Central：反复超时 → 不收录；
- Hugging Face `/api/tags`：401 → 不收录；
- Stack Overflow 必须带 `site=stackoverflow`（旧工具没带，必 400）→ 修复；
- PR/Issues 被禁用的仓库会让周报工具整体 404 → 单数据面容错。

这些失败案例本身就是信任资产——比一百句"稳定可靠"更有说服力。

### 3. 回应 review 用证据

awesome-dsh-plugin 维护者关闭了我们的 PR #352，理由是"纯 CLI 不算插件"。我们没有争论，而是：

1. 按维护者的第二条路加了插件外壳（`dsh.bundle` + `cordis.patch.yml` + 模型工具 `plugin_check`）；
2. 用 CI 证据证明每一条（打包安装 + 真实调用 + 自检）；
3. 回帖附完整验证输出，请维护者重开。

维护者后来在另一个话题里独立验证了我们的工具并确认可用——这就是证据的长期回报。

### 4. 和同赛道工具互相挂链接

我们的 doctor 和另外两个社区诊断工具（moonquake2004/dsh-doctor、dsh-diagnose skill）不竞争，而是三方互补：预防 / 探针 / 症状理解。互相在 README 和讨论里挂链接，约定统一 JSON schema——一个生态的"组织者"比一个孤立工具更容易被记住。

### 5. 把知识沉淀成教程

23 页中英双语教程，全部命令实测：从上手、写插件、发布前体检、peer 版本坑、awesome 榜评审清单、marketplace 发布，到 `undefined.prepare` 崩溃全家桶的调试。教程是复利资产：你睡觉时它还在被人读。

## 给后来者的 checklist

```text
[ ] 找到一个官方讨论里被点名、且你能做好的需求
[ ] 做出可安装产物（npm / github:owner/repo / tarball）
[ ] 每个功能跑真实 API 冒烟，失败案例写进 README
[ ] CI 证明：pack → 全新 profile 安装 → 真实工具调用 → web 启动
[ ] 提交收录（awesome 榜 / marketplace），review 意见用证据回应
[ ] 和同赛道工具互认互链
[ ] 写 2-3 篇中英教程沉淀知识
[ ] 诚实记录未验证的部分
```

## 资源

- 旗舰整合：[dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- 插件医生：[dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 插件搜索：[dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- 模板：[dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- 教程站：https://zoahdev.github.io/dsh-tutorials/
