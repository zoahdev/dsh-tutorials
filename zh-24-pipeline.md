# DeepSeek Harness 插件开发流水线：8 个插件一天上线的实战复盘

> 作者：zoahdev · 2026-08-18 · 全部插件已在 npm/GitHub 上线并通过 CI（Windows Node 24 / pnpm 11 / dsh 0.1.0-rc.6 实测）

# DeepSeek Harness 插件开发流水线：8 个插件一天上线的实战复盘

> 作者：zoahdev · 2026-08-18 · 全部插件均已在 npm/GitHub 上线并通过 CI

## 为什么写这篇

DeepSeek Harness（dsh）开源后生态爆发，`dsh-plugin` 话题下已有数千仓库，但**高质量的验证过的插件**仍然稀缺。这一天我用同一条流水线连发 8 个插件（全部测试 + CI + 真实安装验证 + 收录 PR），把这条流水线和踩过的坑完整记录下来。

## 一句话结论

> 一个 dsh 插件 = 一个 `defineTool` 工具 + 一个 `cordis.patch.yml` + 一套测试/CI + 一份双语 README。质量门槛跑通后，从想法到上线只需要几十分钟。

## 插件结构（最小骨架）

```
my-plugin/
├── package.json          # name/version/description + dsh.bundle.patch
├── cordis.patch.yml      # - insert: - id: xxx  name: my-plugin
├── src/index.ts          # apply(ctx, config) → ctx.tools.register(defineTool({...}))
├── src/version.ts        # 零依赖 peer 守卫（防 pnpm 链错旧 RC）
├── tests/*.spec.ts       # vitest 单测
├── scripts/integration-test.mjs  # 打包后真实安装 + 调用 handler
├── scripts/dsh-smoke.sh  # 全新 profile 安装 + dsh web 启动冒烟
└── .github/workflows/ci.yml
```

关键点：

- **工具定义**：`defineTool` 需要 `parameters`（schema DSL）、`output.schema`（注意：`required` 不在 value schema DSL 支持范围内，输出对象不要写 `required`）、`render`（纯函数把结果渲染成 ContentBlock）、`execute`（返回 canonical JSON，`exec.signal` 处理取消）。
- **类型别名**：返回给模型的对象类型要用 `type` 别名而不是 `interface`，否则 `JsonValue` 索引签名不匹配（TS 报错）。
- **Peer 守卫**：pnpm 可能静默链入旧 RC，运行时用 `satisfiesCaret` 检查 `@deepseek-ai/dsh-tools` 版本，不匹配直接 throw——把静默失败变响亮失败。

## 质量基线（每条线都跑）

```sh
pnpm install
pnpm typecheck     # tsc --noEmit
pnpm build         # tsc → lib/
pnpm test          # vitest（含 mock registry / mock exec 的端到端单测）
pnpm pack
node scripts/integration-test.mjs ./xxx-0.1.0.tgz   # 真实安装 tarball → 加载 → 调用真实 handler → render 断言
bash scripts/dsh-smoke.sh ./xxx-0.1.0.tgz           # 全新 DSH_HOME profile → plugin add → dump-config → dsh web 启动 HTTP 200
```

CI 三件套（每个仓库都配了）：

1. `dsh-plugin-doctor-action` 预检（发布前健康检查）
2. Ubuntu：typecheck → build → test → pack → 打包集成（真实调用工具）
3. Windows：全新 profile 安装 + `dsh web` 启动冒烟（因为官方 npm CLI 缺 linux-x64 pty 预编译，启动冒烟只能在 Windows 跑）

## 8 个插件：每个解决一个真实空位

| 插件 | 解决什么 | 为什么是空位 |
|---|---|---|
| dsh-dep-audit | 依赖供应链卫生审计（peer 可解析、dist-tag 矛盾、过期、许可证、漂移） | 注册表安全类只有 poison-guard 一个；实测抓到 `@deepseek-ai/dsh-tools` 的 `latest=0.0.1-rc.1` 与声明范围矛盾（#2763 类） |
| dsh-llms-forge | 从 package.json + README 生成 llms.txt | llms.txt 方向零命中；AI agent 进仓库最先找它 |
| dsh-cn-boot | 国内网络引导（探测 npm/GitHub/HF/代理 + 镜像推荐） | 零命中；本机实跑准确抓到 HuggingFace 超时 |
| dsh-timesheet | 从会话日志做基于 turn 的时间跟踪 | 时间统计零命中（token 仪表盘满地，没人统计墙钟时间）；实跑 52 turns / 4h56m |
| dsh-discussions-radar | 官方 Discussions 雷达 | 官方只用 Discussions，但没插件把它暴露给 agent |
| dsh-readme-forge | 生成 README.md（package.json + cordis.patch.yml + 源码布局） | README 生成零命中；与 llms-forge 组成管道 |
| dsh-firstrun | 首次运行体检（工具链/profile/API Key/工作区/注册表 + 下一步） | onboarding/quickstart 零命中 |
| dsh-disk-audit | 磁盘占用审计（会话日志能涨到数百 MB） | 存储审计零命中 |

**方法论**：每次动手前先扫 dsh-subscribe 注册表（916 插件）确认空位，避免撞车。比如 vault 方向看到 dsh-vault 已迭代到 v1.8.1（393 测试）就果断放弃。

## 踩坑记录（都是真金）

1. **npm 撞名**：`dsh-quickstart` 已被占用（0.2.0）→ 全套改名 `dsh-firstrun`（包名/仓库/文档/CI/脚本一起改，否则 CI 会挂）。
2. **Windows shim**：`spawnSync('pnpm', args)` 在 Windows 上找不到 pnpm（它是 .cmd shim）→ win32 走命令串 + shell:true + 引号助手。
3. **CI 漏改**：改名时 grep 模式没同步，Windows 冒烟红了一次 → grep 改为新 id 后重跑全绿。
4. **0xsline 收录方式**：维护者反馈 CATALOG.md 是 CI 自动生成、手改会被覆盖 → 正确入口是 README.md + README.zh-CN.md 双文件。
5. **dsh-tools 的 latest dist-tag 是坏的**（0.0.1-rc.1 vs 声明 ^0.1.0-rc.6）——这是生态级 ERESOLVE 的根因之一（#2763），插件开发时要用 `@next`/rc.6。

## 社区贡献闭环

发布 ≠ 结束。完整闭环：

1. **官方 Show Your Plugins**：一个帖子把家族串起来（#3123），每次更新追加评论而不是刷新帖。
2. **Q&A 用证据回复**：#55（dsh 全局安装找不到 cordis-plugin-timer）——先核实 npm 元数据 + 本地 require.resolve 实验，再给出有证据的排查回复（原回帖者只说“已解决”没说方案）。
3. **收录 PR**：0xsline（README 双文件）+ awesome-dsh-plugin（data/plugins yml + 自动生成 README），1 天 gate 后转绿。
4. **自己的注册表/生态目录**：dsh-subscribe（916 插件 / verified 29）+ dsh-ecosystem 同步 ✅。

## 给新插件作者的建议

- **先扫空位再动手**：注册表 + 官方 Discussions 的 Ideas 分类。
- **零依赖优先**：能用 node: 内置就用内置（fetch/spawnSync/fs），发布即装即用。
- **默认只读**：写盘/改配置一律显式 opt-in，这是 dsh 社区最看重的信任基线。
- **绝不在输出里打印密钥值**：只显示变量名。
- **双语 README + llms.txt**：AI 可读性就是被发现率。

## 链接

- 8 个插件仓库：github.com/zoahdev/dsh-dep-audit · dsh-llms-forge · dsh-cn-boot · dsh-timesheet · dsh-discussions-radar · dsh-readme-forge · dsh-firstrun · dsh-disk-audit
- 官方展示帖：https://github.com/deepseek-ai/deepseek-harness/discussions/3123
- 生态地图：https://github.com/zoahdev/dsh-ecosystem

