# 通过 awesome-dsh-plugin 收录 gate（Submission gate 完全指南）

> 适用：任何想把自己的 dsh 插件收录进 [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) 目录的作者。本文所有行为均经实测（2026-08-19，repo 年龄 gate 场景）。

## 1. Gate 检查什么

收录 PR 的 `Submission gate` 检查三件事（`scripts/check-submission.mjs` + `pr-gate.yml`）：

1. **`dsh.bundle` manifest 存在**——你的仓库/包必须声明 bundle（`dsh.bundle` 字段），否则直接拒绝；
2. **仓库年龄 ≥ 1 天**（从 repo `created_at` 算，PR 创建时间不算）；
3. **仓库 commit 数 ≥ 10**。

三个都过 → 绿色 `success`；任何一个不过 → 红色 `failure`，PR 显示 `mergeStateStatus: UNSTABLE`（可合并但检查失败）。

## 2. 最常见的红灯：年龄不足

新仓库提收录 PR 时几乎必中 `repository is X.X days old (needs 1) — resubmit in about 15h`。这是**有意设计**（防止一次性账号刷收录），不是你的提交有问题。

## 3. 到点后怎么转绿（关键机制）

gate 判定是**快照**：年龄满足后不会自己刷新，需要一次新的 push/检查触发。两个途径：

**A. 官方自动兜底（什么都不用做）**

仓库的 `regate.yml` 每 6 小时（cron 在 :19 UTC）自动重跑所有"aged in"的 PR（判定失败原因是 `days old|commit(s)` 的自动重试），到点后 6 小时内自动转绿。**唯一要做的就是等。**

**B. 手动提前（想最快变绿时）**

往 PR 分支推一个空提交戳即可触发 pr-check → gate 重评：

```bash
git checkout <pr-branch>
git commit --allow-empty -m "chore: gate re-trigger stamp"
git push origin <pr-branch>
```

⚠️ **不要尝试用 API 重跑 workflow**：`POST /actions/runs/{id}/rerun` 会返回 `403 Must have admin rights to Repository`（org 仓库只有管理员能 rerun），自己 fork 推提交才是有效路径。

## 4. 其它常见红灯

| 症状 | 原因 | 处理 |
|---|---|---|
| `dsh.bundle` not declared | 包 manifest 缺 `dsh.bundle` 字段 | 补上 bundle/patch 声明 |
| `repository is 0.1 days old` | 年龄 gate | 等 1 天（regate 自动处理） |
| `commit(s)` 不足 | repo < 10 commits | 补真实提交（README/SECURITY/CHANGELOG/CONTRIBUTING 都算），**别刷空提交充数** |
| `check`（PR check）失败 | 条目格式/内容问题 | 读该检查的输出修到绿（gate 不重跑内容错的 PR，regate 只处理年龄/commit 类） |

## 5. 我们的实测数据

- 8 个仓库（每个 10 commits）全部一次通过 commit 检查，唯一红灯是年龄；
- 到点时间 = `repo created_at + 24h`，精确到分钟（GitHub API 的 created_at）；
- 手动戳提交后 gate 在下一次 pr-check 完成后转绿；官方 regate 也会在 6h 窗口内兜底。

## 6. 检查你的到点时间

```bash
gh api repos/<owner>/<repo> --jq .created_at   # repo 创建时间
# 到点 = created_at + 24h（UTC）
```

收录后记得更新你的 README 徽章/状态，并在官方 Discussions 展示帖（如 #3123 模式）同步进度。