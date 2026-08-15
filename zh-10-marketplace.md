# 在 DSH Marketplace 发布你的插件

> 作者：zoahdev · 2026-08-15 · 基于真实提交（ydhrdh/dsh-marketplace PR #1）与实测

## 什么是 DSH Marketplace

[dsh-marketplace](https://github.com/ydhrdh/dsh-marketplace)（讨论 [#1752](https://github.com/deepseek-ai/deepseek-harness/discussions/1752)）是一个开源的插件市场：每个插件一个 JSON 清单，PR 审核制，Git 历史即审计日志，前端静态部署零成本。收录后用户可以在网页上浏览、搜索、一键复制安装命令。

## 收录条件

- 插件**公开可安装**：npm 包（target: `npm`）、Git 仓库（target: `git`，如 `github:owner/repo`）或本地路径（target: `local`）；
- 清单通过 `npm run validate`；
- `install.command` 必须带 `{profile}` 占位符；
- 仓库打上 [`dsh-plugin`](https://github.com/topics/dsh-plugin) topic。

## plugin.json 长什么样

必需字段：`id`（与目录名一致）、`name`、`version`（semver）、`description`、`author{name,url}`、`license`、`category`（skills/tools/web/infrastructure/integration/workflow/experimental/other）、`install{target,spec,command}`、`repository`。`verified` 由维护者审核后置 true，首次提交写 `false`。

```json
{
  "id": "dsh-plugin-doctor",
  "name": "Plugin Doctor",
  "version": "1.3.0",
  "category": "tools",
  "install": {
    "target": "git",
    "spec": "github:zoahdev/dsh-plugin-doctor",
    "command": "dsh plugin --profile {profile} add github:zoahdev/dsh-plugin-doctor"
  },
  "repository": "https://github.com/zoahdev/dsh-plugin-doctor"
}
```

## 提交流程

```sh
git clone https://github.com/ydhrdh/dsh-marketplace
mkdir -p registry/plugins/<your-plugin-id>
# 写 plugin.json
npm install
npm run validate   # 先本地过
npm run build      # 重新生成索引
# 开 PR，CI 必须绿
```

## 实测踩坑：git 安装会被 allowBuilds 拦

用 `github:owner/repo` 安装插件时，pnpm 11 默认禁止 git 依赖执行 `prepare` 构建脚本，第一次 `dsh plugin add` 会报：

```text
[ERR_PNPM_GIT_DEP_PREPARE_NOT_ALLOWED] ... not in the "allowBuilds" allowlist
```

dsh CLI 会打印精确指引。把对应 key 加进 profile 的 `pnpm-workspace.yaml`，再跑一次即可：

```yaml
allowBuilds:
  dsh-plugin-doctor@https://codeload.github.com/zoahdev/dsh-plugin-doctor/tar.gz/<sha>: true
```

**建议**：PR 描述里附上你实测的安装输出（成功 + 失败各一段），维护者审核时直接照做，能显著加快合并——我们 PR #1 就是这么写的。

## 参考实例

- 我们的收录 PR：https://github.com/ydhrdh/dsh-marketplace/pull/1（5 个插件：intelligence/doctor/search/radar/template）
- 市场仓库：https://github.com/ydhrdh/dsh-marketplace
- 在线市场：https://ydhrdh.github.io/dsh-marketplace/
