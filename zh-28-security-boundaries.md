# 沙箱信任边界与 fail-closed 原则（Security Boundaries）

> 适用：部署 dsh 的运维、写插件的作者、做安全评审的人。本章把社区 2026-08 的三次安全讨论（#3245/#3212/#3223）沉淀成可操作的检查清单。全部代码引用对照官方 main HEAD `99f6f02`（rc.7）。

## 1. 一句话原则

**信任边界必须可强制，不能靠声明。** 文档写"这是隔离"不算隔离；沙箱说"read-only"就必须所有执行路径都真的只读。任何一个绕过点都会让整条边界失效。

## 2. 案例：#3245 run_code 沙箱逃逸（Critical）

### 发生了什么

dsh 的普通执行路径都走文件沙箱：
- bash → `bash-sandbox` 的 `ctx.sandbox.confine(...)`（Landlock/bwrap/Seatbelt）；
- 文件工具 → `fs-sandbox` 的围栏（read-only 拒绝一切变更，workspace-write 限工作区内）。

但 **Code Mode 的 `run_code`** 走第三条路：`code-runtime-worker-thread` 用 `new Worker(path, { env: {}, execArgv: [] })` 启动 Node worker，模型代码经 `new AsyncFunction(...)` 执行——**没有任何文件效应策略**（无 `ctx.sandbox.confine`、无 `--experimental-permission`）。worker 与宿主进程同 UID、同内核，模型程序可以 `await import('node:fs')` 读写任意文件、`await import('node:child_process')` 起任意无约束子进程。

默认沙箱是 read-only（`sandbox-policy` 默认值），所以"Code Mode + 默认沙箱"= 完全逃逸，CVSS 10.0/9.8。

### 教训一：枚举所有执行路径，逐个对照边界

检查一个沙箱时，不要只看"有没有沙箱"，要看**每一条能执行代码的路径**是否都经过同一个策略面：

| 路径 | 是否受沙箱约束（rc.7 修复前） |
|---|---|
| bash / pwsh | ✅ `ctx.sandbox.confine` |
| fs 工具 | ✅ 围栏 + 路径规范化 |
| run_code (worker-thread) | ❌ 无文件效应策略 |
| workflow worker-thread (node:vm) | ❌ 同类无约束面 |
| cordis-host-runner 动态插件域 | ❌ 文档明示 not containment |

### 教训二：fail-closed 优于 fail-open

在根修（把 worker 纳入 `ctx.sandbox.confine`）落地前，正确的最小缓解是 **fail-closed**：worker-thread isolation + 受限策略（read-only/workspace-write）→ 拒绝执行，而不是静默放行。已落地分支 `fix/run-code-failclosed-sandbox`：dispatch 前解析沙箱策略，`runtime.isolation === 'worker-thread' && policyMode !== 'danger-full-access'` → 抛错（提示三条出路：DSH_TOOLS_MODE=native / danger-full-access / 挂可约束 runtime）。

## 3. 案例：#3212 允许清单 > 禁止清单

dsh-crew 讨论里提炼的原则：**文本安全带是 policy，不是安全边界**。让 agent"不要执行危险命令"的提示，和沙箱"不允许执行"是两回事——前者可被绕过（echo/命令行/MCP 都能绕开文本规则），后者是内核强制。

检查你的部署：如果某个安全保证只存在于 system prompt 或插件描述里，它就不是边界；要把它变成沙箱策略/权限枚举/审批门。

## 4. 案例：#3223 attestation 锚点（可验证性）

远程部署的信任边界（#3209/#3210/#3211 三部曲）核心是**可验证而非可声明**：
- externalUrl=说对地址、Tailscale-User-Login=说对身份、surface=装到一起——都不改 trust fence；
- ToolRuntime 的 attestation/settlement 锚点（#3223）的"词汇"应统一到 CHA2A（#3192），避免第三种方言。

给运维的自查：你的"可信来源"是**进程强制**（签名验证、令牌校验）还是**约定俗成**（文档说"只从内网访问"）？后者在威胁模型里不算控制。

## 5. 一张可复用的检查表

做任何 dsh 安全评审时：

1. **列执行面**：bash/fs/run_code/workflow/cordis-runner/MCP——每条路径落到策略面，标注 ✅/❌；
2. **找绕过**：对每个 ❌ 问"默认配置下可达吗？"（#3245 就是"Code Mode 必须显式开，但开了就是默认可达"）；
3. **定 fail-closed**：无法立即根修的，先让"受限策略 × 无约束路径"组合**报错**而不是放行；
4. **查文档诚实度**：README/Agent Note 写的"containment, not a security boundary"要如实翻译成用户能懂的信任等级（worker-thread ≈ danger-full-access）；
5. **回归测试**：每个边界配一个"逃逸尝试"用例（#3245 补丁的测试就是 read-only + worker-thread → 必须拒绝）。

## 6. 相关社区资产

- 家族图谱家族 13「无约束执行面绕过沙箱」：dsh-ecosystem/docs/bug-families.md
- 补丁队列 #48（#3245 fail-closed）：dsh-docs/docs/specs/upstream-patches.md
- 教程 zh-25：社区贡献工作流（怎么把这类发现写成证据级回复）
- 教程 zh-27：家族图谱 triage（新 bug 先查族）
