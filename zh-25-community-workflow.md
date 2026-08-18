# dsh 社区贡献工作流：从 bug 报告到证据级回复

> 适用：想在 dsh 官方 Discussions 里持续输出高质量贡献的人。以下全部来自 2026-08-18~19 的实战（7 条证据级回复 + 1 个值班脚本 + 2 个修复日工具），命令实测于 Windows / Node 24 / gh CLI。

## 1. 值班：先扫一遍"谁还没人答"

官方讨论区每天几十条新帖，先跑值班脚本，别靠肉眼翻：

```bash
node dsh-ecosystem/scripts/discussion-triage.mjs --since 24
```

单次 GraphQL 拉最新 100 帖 + 评论作者，输出四列：# / comments / 我方已答(✅) / title，末尾列出 **0 回复候选**。要点：

- `--since 24`（小时）、`--all`（不过滤）、`--json`（机器可读）
- "ours" 按当前 gh 登录名判定（`gh api user --jq .login`），答过自动跳过
- 值班纪律：同一帖**只答一次**；帖主已自带完整根因时，只做**核验 + 增量**，不重复

## 2. 核验：回复前先对照当前 main

对任何带"源码定位"的帖子，先核验再说话，三步：

```bash
# 1) 浅克隆官方仓库（一次）
git clone --depth 1 https://github.com/deepseek-ai/deepseek-harness.git

# 2) 记下当前 HEAD，回复里写清楚核验基线
git log -1 --format="%H %s"

# 3) 用 rg 按关键词/行号定位，逐行确认
rg -n "listArtifacts|spillAll|SIGINT" packages/session/session-persistence-jsonl/src packages/subprocess/subprocess-local/src apps/cli/src
```

典型收益（实战例子）：

| 帖子 | 核验结论 |
|---|---|
| #675 SQLite torn-tail | 帖主引用的行号在当前 main 逐行成立 → 补 tornFrom 唯一消费方 + 回归测试建议 |
| #1047 session.list 500 | 成立且 rc.6→rc.7 未修 → 补同循环另外两个未隔离抛错点 |
| #3190 spill ENOENT | 成立 → 指出仓库已有 spillDisabled+discardSpill 降级模式可复用，给最小补丁 |
| #3195 CTRL_C_EVENT | 成立且未修 → 补 SIGBREAK 同通道风险 + Node 层无 CREATE_NEW_PROCESS_GROUP 的约束 |

"核验成立 + 未修"是对维护者最有用的信息：说明报告可信、优先级值得评估，且给出精确基线。

## 3. 实证：能本地跑的断言就地跑

源码引用之外，能复现的跑一遍再写。实战例：CJK 检索帖（#3206）声称 unicode61 对中文不可用、trigram 可命中，用 Node 24 内置 node:sqlite 直接验证：

```js
import { DatabaseSync } from 'node:sqlite'
const db = new DatabaseSync(':memory:')
db.exec("CREATE VIRTUAL TABLE t USING fts5(content, tokenize='unicode61')")
db.exec("CREATE VIRTUAL TABLE t2 USING fts5(content, tokenize='trigram')")
db.prepare('INSERT INTO t(content) VALUES (?)').run('索引优化减少Token消耗的句子')
db.prepare('INSERT INTO t2(content) VALUES (?)').run('索引优化减少Token消耗的句子')
console.log(db.prepare('SELECT count(*) c FROM t WHERE t MATCH ?').get('Token消耗').c)  // 0
console.log(db.prepare('SELECT count(*) c FROM t2 WHERE t2 MATCH ?').get('Token消耗').c) // 1
```

顺带钉出帖主表格没展示的边界：**2 字 CJK 查询（如"句子"）两条词法臂都返回 0**（trigram 至少 3 字符）。带实测数据的评论比纯推理高一个量级。

## 4. 回复：走 GraphQL，别走 REST

官方 Discussions 的评论创建对普通 token 走 REST 会 404（GET 正常、POST 不行），直接 GraphQL：

```bash
# 取 discussion node id
gh api repos/deepseek-ai/deepseek-harness/discussions/3174 --jq .node_id
```

```json
{
  "query": "mutation AddComment($discussionId: ID!, $body: String!) { addDiscussionComment(input: {discussionId: $discussionId, body: $body}) { comment { id url } } }",
  "variables": { "discussionId": "D_kwD...", "body": "正文（Markdown）" }
}
```

```bash
# 用 gh api graphql --input - 提交（避免 shell 转义炸掉正文）
```

## 5. 家族化：同根因多报合并，帮维护者排优先级

实战两例：

- **#3190（spill 目录 ENOENT 崩溃）+ #3203（windows-acl 沙箱临时目录被清理）** = "外部 OS 临时目录清理破坏运行中 dsh" 一个家族。共同根因：运行期持有的进程私有 %TEMP% 目录对外部清理无防护。修法统一为"在目录被消费的边界遇缺失即重建/降级"。
- **#675（单行坏 → 连坐删除有效行）+ #1047（单文件坏 → 连坐清空列表）** = "坏工件隔离" 一个家族。建议统一回归测试约定：构造单个坏工件，断言其余工件照常可用。

在回复里显式链接同族帖（"#1047 与 #675 是同一家族"），维护者合并处理时一眼能看到全景。

## 6. 刷屏纪律

- 0 回复帖优先；已有高质量回复的帖子不重复
- 帖主已带完整修复方案的，回复定位为"核验 + 增量"，别抢戏
- 每条回复只写有依据的内容；推测部分明确标注"待验证"
- 中文帖回中文、英文帖回英文；正文给行号/HEAD/测试输出，可复核

## 7. 配套工具清单

| 工具 | 位置 | 用途 |
|---|---|---|
| discussion-triage.mjs | dsh-ecosystem/scripts | 每晚扫值班队列（单请求，标记已答） |
| verify-2763-fix.mjs | dsh-ecosystem/scripts | #2763 修复日自动重扫 + 重评 + 出报告 |
| full-registry-scan.mjs | dsh-ecosystem/scripts | 全注册表供应链扫描（325 插件） |
| restore-publish.ps1 | 本地 work/ | 收录 PR gate 到点后重触发（step 7/8） |

> 官方 awesome-dsh-plugin 的 Submission gate 会自动重跑"aged in"的 PR（regate.yml 每 6h 一次），到点后即使不手动操作也会转绿；主动重触发只是为了最早变绿。