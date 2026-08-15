# 让插件通过 awesome-dsh-plugin 的 Review：维护者 Checklist

> 作者：zoahdev · 2026-08-15 · 依据 awesome-dsh-plugin 维护者 fkysly 在 PR #352 的真实评审

## 收录边界（先搞清楚）

[awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) 只收录 **`dsh plugin add` 一行能装进 DeepSeek Harness 的插件**（边界见 [PR #248](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/248)）。

纯 CLI 工具不在收录范围内——除非发到 npm 用 `npx` 分发。维护者原话：**"一个能被 agent 调用的插件体检器，对整个生态的价值比 CLI 大得多"**。所以：要么是插件，要么别投这个榜单。

## 逐项 Checklist

| # | 检查项 | 不过的样子 | 自查方式 |
|---|---|---|---|
| 1 | `package.json` 有 `dsh.bundle` | 只有 `bin`，没有 `dsh` 字段 | `node -e "console.log(require('./package.json').dsh)"` |
| 2 | `dsh.bundle.patch` 指向存在的 `cordis.patch.yml` | 装完 dump-config 没有你的插件 | `dsh --profile web --dump-config` |
| 3 | `cordis.patch.yml` 合法、有带 `id` 的 `insert` | YAML 报错 / 空列表 | doctor 的 `patch` 检查 |
| 4 | `files` 白名单包含构建产物 | 装到别人机器只有 `src/` | `pnpm pack --dry-run` |
| 5 | `prepare` 脚本 | git 安装不构建 | `npm pkg get scripts.prepare` |
| 6 | 至少一个模型工具（agent 能调） | 装了但 agent 用不了 | 插件 `apply()` 里 `ctx.tools.register` |
| 7 | CI 证明"真实可调用" | 只证明"能加载" | CI 里装 tarball → 注册 → 真实执行 handler → 断言 |
| 8 | README 说明安装方式与用法 | 用户不知道装完干嘛 | 照 README 自己走一遍 |
| 9 | 真实安装验证 | 本地没装过 | `dsh plugin --profile web add <tarball>` |

## 一条命令跑完 80% 的检查

```sh
git clone https://github.com/zoahdev/dsh-plugin-doctor.git
cd dsh-plugin-doctor && pnpm install && pnpm build
node lib/bin.js --build --full /path/to/your-plugin
```

输出覆盖：manifest / patch / entry / files / build / pack / 全新 DSH profile 安装 / dump-config 含插件 id。

## 最常被退回的三个原因（维护者原话）

1. **缺 `dsh.bundle`** —— 只有 `bin`，没有插件外壳；
2. **`files` 只列了 `src/`** —— 构建产物 `lib/` 没进白名单，也没有 `prepare`；
3. **纯 CLI 当插件投** —— 装不上，也不属于榜单边界。

## 被拒之后怎么办

维护者关闭 PR 不等于项目不行。以 [PR #352](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin/pull/352) 为例：

1. 维护者给了两条路（纯 CLI → npm `npx`；或加插件外壳）；
2. 我们选了"插件外壳"：`cordis.patch.yml` + `dsh.bundle` + 一个 `plugin_check` 模型工具，`bin` 保留；
3. 用真实 CI（打包安装 + 真实调用 + 自检）证明每一条，再回帖请维护者重开。

把评审意见当验收标准，而不是争论点。

## 相关资源

- 榜单：[awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin)
- 自检工具：[dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 模板：[dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
