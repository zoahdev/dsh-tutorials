# dsh 插件依赖陷阱：prerelease peer 版本与 ERESOLVE

> 作者：zoahdev · 2026-08-15 · 基于 dsh 0.1.0-rc.6 实测（npm / pnpm 行为已实际验证）

## 一个没人讲的 semver 事实

`^0.1.0-rc.6` 的含义：

```text
>=0.1.0-rc.6 且 <0.2.0
```

它**匹配**：`0.1.0-rc.6`、`0.1.0-rc.7`、`0.1.0`、`0.1.1`……
它**不匹配**：`0.1.0-rc.3`、`0.0.9`、`0.2.0`。

所以把 peer 写成 `^0.1.0-rc.6` 不是"锁定"，只是一个下限。**如果你宿主里已经装了 `0.1.0-rc.3`，插件要求 rc.6，两者直接冲突。**

## 三种安装器，三种表现

| 场景 | npm | pnpm（默认） |
|---|---|---|
| 宿主已有旧 RC `0.1.0-rc.3` | `ERESOLVE` peer dependency conflict，直接失败 | 可能只给一条泛泛警告，继续装 |
| 宿主没有 dsh-tools | 尝试装 peer | `autoInstallPeers=false` 时**不装**，运行时才炸 |
| 宿主是 rc.6 | 正常 | 正常 |

pnpm 的"静默"最坑：插件 `apply()` 里 `import { defineTool }` 链到了旧版，如果旧版没有新 API，报错出现在 agent 第一次调工具时，而不是安装时。

## 2026-08-15 实测：npm 的 latest 标签本身就是个坑

`@deepseek-ai/dsh-tools` 在 npm 上的 dist-tags 实测：

```text
latest: 0.0.1-rc.1
next:   0.1.0-rc.6
```

也就是说，**裸装** `npm install @deepseek-ai/dsh-tools`（或 pnpm 因 peer 自动安装）拿到的是 `0.0.1-rc.1`，而不是我们一直在用的 `0.1.0-rc.6`——`0.0.x` 根本不满足插件 peer 的 `^0.1.0-rc.6`，这就是大量 ERESOLVE / peer conflict 的真正来源之一。

插件/宿主都要显式装：

```sh
pnpm add @deepseek-ai/dsh-tools@next   # 0.1.0-rc.6
# 或统一用 dsh 官方渠道安装 dsh@0.1.0-rc.6
```

## 三层防护（模板怎么做的）

1. **peer 声明**：只写实测过的范围 `^0.1.0-rc.6`，不假装兼容更宽；
2. **运行时守卫**：`apply()` 里解析实际链到的 `@deepseek-ai/dsh-tools` 版本，不满足就抛出带指引的明确错误（见 [dsh-plugin-template/src/version.ts](https://github.com/zoahdev/dsh-plugin-template)）；
3. **安装验证**：CI 里把 tarball 装进全新宿主，跑真实工具调用（[dsh-plugin-template 的 integration-test](https://github.com/zoahdev/dsh-plugin-template)）。

## 用户侧：遇到 ERESOLVE / peer conflict 怎么办

```sh
# 1. 先看宿主里实际是什么版本
pnpm why @deepseek-ai/dsh-tools
# 或
npm ls @deepseek-ai/dsh-tools

# 2. 把宿主升级到插件验证过的版本（0.1.0-rc.6），再装插件
dsh upgrade   # 或按官方发布渠道升级
dsh plugin --profile web add <你的插件>

# 3. 确认真的解析对了
dsh --profile web --dump-config
```

**不要**用 `--legacy-peer-deps` 绕过：那是"假装没冲突"，插件运行时大概率照炸。

## 插件作者侧：发布前必查

```sh
node /path/to/dsh-plugin-doctor/lib/bin.js --full .
```

`install` 检查会真实执行 `dsh plugin add` + `--dump-config`；再配合全新宿主里的真实工具调用，才算验证过 peer 兼容性。

## 相关资源

- 模板（含守卫源码）：[dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- 自检工具：[dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 官方插件发布指南：[publish.md](https://github.com/deepseek-ai/deepseek-harness/blob/HEAD/docs/user/develop/basic/publish.md)
