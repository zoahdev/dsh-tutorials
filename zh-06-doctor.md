# 用 dsh-plugin-doctor 给你的插件做体检

> 作者：zoahdev · 2026-08-15 · 全部命令实测于 Windows（Node 24 / pnpm 11）

## 目录

- [为什么需要体检](#为什么需要体检)
- [快速开始](#快速开始)
- [检查项](#检查项)
- [完整体检：--full](#完整体检--full)
- [接入 CI](#接入-ci)
- [常见问题](#常见问题)

## 为什么需要体检

"插件能加载"不等于"插件能用"。两个真实坑：

1. **manifest 结构错**：`dsh.bundle`、`cordis.patch.yml`、`prepare` 少一个，别人一装就炸；
2. **peer 版本被静默链错**：pnpm 默认配置可能把旧的 `@deepseek-ai/dsh-tools` RC 链进你的插件（实测 `0.1.0-rc.3` 被链给 `^0.1.0-rc.6`，只有一条泛泛警告，不报错）。

[dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor) 把这两类问题变成"本地一跑就知道"。

## 快速开始

```sh
git clone https://github.com/zoahdev/dsh-plugin-doctor.git
cd dsh-plugin-doctor
pnpm install
pnpm build

# 对目标插件目录做快速体检
node lib/bin.js /path/to/my-plugin
```

输出示例：

```text
[PASS] manifest: my-plugin@1.0.0 bundle manifest ok
[PASS] patch: 1 plugin row(s): my-hello
[WARN] entry: entry lib/index.js not built yet (run pnpm install && pnpm run build)
[PASS] files: ships: lib, cordis.patch.yml
✅ ALL CHECKS PASSED
```

退出码：全部通过为 `0`，否则为 `1`。

## 检查项

| 检查 | 验证内容 | 默认 |
|---|---|---|
| `manifest` | `package.json` + `dsh.bundle` + `dsh.bundle.patch` + `prepare` + `main` | ✅ |
| `patch` | `cordis.patch.yml` 合法且至少一条带 `id` 的 `insert` | ✅ |
| `entry` | `main` 指向的文件存在 | ✅ |
| `files` | 声明了 `files` 白名单 | ✅ |
| `build` | `pnpm run build` 真实执行 | `--build` |
| `pack`+`install` | 打包 → 装进全新 DSH profile → `--dump-config` 确认插件 id | `--full` |

## 完整体检：--full

```sh
node lib/bin.js --full /path/to/my-plugin
```

这是最接近"别人机器上安装"的本地模拟：

```text
[PASS] manifest: dsh-plugin-template@0.1.0 bundle manifest ok
[PASS] patch: 1 plugin row(s): template-hello
[PASS] entry: entry lib/index.js exists
[PASS] files: ships: lib, cordis.patch.yml, README.md, LICENSE
[PASS] pack: packed dsh-plugin-template-0.1.0.tgz
[PASS] install: plugin id(s) present in composed config: template-hello
✅ ALL CHECKS PASSED
```

## 插件外壳（v1.1.0）：让 agent 直接体检

从 v1.1.0 起，dsh-plugin-doctor 自带插件外壳（`dsh.bundle` + `cordis.patch.yml`），可以直接装进 DSH：

```sh
dsh plugin --profile web add dsh-plugin-doctor
# 或本地 tarball：
dsh plugin --profile web add ./dsh-plugin-doctor-1.1.0.tgz
```

装完在 DSH 里对 agent 说：

> 检查一下这个插件能不能发布 —— 先跑 build，再做完整验证。

agent 会调用 `plugin_check` 工具（`dir` + 可选 `build`/`full`），逐项返回 PASS/WARN/FAIL 和整体 `ok`，不用切出 DSH。

## 接入 CI

```sh
node lib/bin.js --json ./my-plugin
```

`--json` 输出机器可读报告 + 退出码，直接塞进 GitHub Actions：

```yaml
- run: node /path/to/dsh-plugin-doctor/lib/bin.js --build --json ./my-plugin
```

## 常见问题

**Q：`entry` 报 WARN？** 插件还没构建：先 `pnpm install && pnpm run build`。

**Q：`manifest` 报 FAIL？** 对照[官方发布指南](https://github.com/deepseek-ai/deepseek-harness/blob/HEAD/docs/user/develop/basic/publish.md)补 `dsh.bundle` 与 `prepare`。

**Q：`install` 报 FAIL？** 插件 id 没进组合配置，多半是 `cordis.patch.yml` 的 `id` 与期望不符，或 peer 版本不兼容（升级宿主到 0.1.0-rc.6+ 再试）。

## 相关资源

- 仓库：[zoahdev/dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- 配套模板：[zoahdev/dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)（CI 已内置真实工具调用冒烟）
- 提案来源：[RFC #1629](https://github.com/deepseek-ai/deepseek-harness/discussions/1629)
