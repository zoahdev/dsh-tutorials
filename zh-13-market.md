# 在 DSH 里一键装插件：dsh-subscribe in-harness 市场实战

> 作者：zoahdev · 2026-08-15 · 对应仓库 https://github.com/zoahdev/dsh-subscribe（v0.3.0）

## 一句话

把插件市场装进 DeepSeek Harness：在浏览器里打开 `/dsh-subscribe/`，浏览 536 个插件，点一下按钮就能安装、卸载、更新、批准构建脚本——每个按钮都真实驱动本机 dsh CLI。

## 1. 安装市场插件

```sh
dsh plugin --profile web add github:zoahdev/dsh-subscribe#path:/plugin
```

重启 `dsh web`，然后打开：

```text
http://localhost:<你的 dsh 端口>/dsh-subscribe/
```

## 2. 页面能干什么

- **浏览/搜索**：536 插件，中英双语描述，按星标或"已验证"排序；
- **一键安装**：点 Install，页面显示实时进度，装完自动刷新已安装列表；
- **一键卸载 / 更新**：已安装卡片上直接操作；
- **批准构建脚本**：git 插件被 pnpm `allowBuilds` 拦截时，页面会提示一键批准（只允许已安装的包）；
- **Steam 式订阅**：把插件加进订阅清单 → 导出 `subscriptions.json` → 终端跑同步命令，一次装齐；
- **日志导出**：`/dsh-subscribe/logs` 下载脱敏文本日志，报 bug 时直接贴。

## 3. 底层是什么

插件在宿主 webServer 上挂了一组同源 HTTP API：

| 路由 | 方法 | 作用 |
| --- | --- | --- |
| `/dsh-subscribe/registry` | GET | 注册表 + 统计 |
| `/dsh-subscribe/installed` | GET | 已安装列表 |
| `/dsh-subscribe/status` | GET | 实时安装进度 |
| `/dsh-subscribe/updates` | GET | 更新检测（npm latest / git HEAD） |
| `/dsh-subscribe/install` | POST | 一键安装（curated 或 file:/link:） |
| `/dsh-subscribe/uninstall` | POST | 一键卸载 |
| `/dsh-subscribe/update` | POST | 一键更新 |
| `/dsh-subscribe/approve-builds` | POST | 批准构建脚本 |
| `/dsh-subscribe/cancel` | POST | 取消进行中的操作 |
| `/dsh-subscribe/logs` | GET | 脱敏日志导出 |

安全模型：写操作必须同源 POST；安装 spec 只能来自 curated 注册表或显式 `file:`/`link:`；spawn 目标走白名单正则。

## 4. 验证证据（不是"能加载"）

CI 与本地实测完整链路：

```text
全新 DSH profile → 安装 tarball → --dump-config 含 dsh-subscribe
→ dsh web 启动 HTTP 200
→ GET /dsh-subscribe/registry → 536 plugins, 20 verified
→ POST /dsh-subscribe/install（真实执行 dsh CLI）→ exit 0
→ GET /dsh-subscribe/installed → 包含 dsh-subscribe
```

## 5. 命令行版（不想开网页时）

```sh
node scripts/dsh-subscribe.mjs list                 # 列出插件
node scripts/dsh-subscribe.mjs install <id>         # 直接装一个
node scripts/dsh-subscribe.mjs sync --dry-run       # 预览订阅同步
node scripts/dsh-subscribe.mjs sync --profile web   # 装齐全部订阅
```

## 相关

- 网页商店（不装插件也能用）：https://zoahdev.github.io/dsh-subscribe/
- 对话内 agent 工具：`market_search` / `market_stats` / `market_install_command`
