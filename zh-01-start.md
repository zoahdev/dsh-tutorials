# 从零上手 DeepSeek Harness：安装、启动与第一次对话

> 作者：zoahdev · 2026-08-15 · 本教程所有命令均在本机（Windows + Node 24 + pnpm 11）实测

## 目录

- [DeepSeek Harness 是什么](#deepseek-harness-是什么)
- [环境要求](#环境要求)
- [方式一：npm 一行命令启动（推荐新手）](#方式一npm-一行命令启动推荐新手)
- [方式二：从源码运行](#方式二从源码运行)
- [第一次对话：配置模型](#第一次对话配置模型)
- [Web UI 速览](#web-ui-速览)
- [常见坑（实测踩过）](#常见坑实测踩过)
- [FAQ](#faq)

## DeepSeek Harness 是什么

DeepSeek Harness（简称 `dsh`）是 DeepSeek 在 2026-08-13 以 MIT 协议开源的首款 Agent 产品：一个 **agent harness（智能体框架）**。它不是一个模型，而是让模型使用工具、执行任务、与开发者协作的基础设施。它的核心设计是「一切皆插件」，由 Cordis 驱动。

> 仓库：https://github.com/deepseek-ai/deepseek-harness
> 当前状态：开发者预览 v0.1，迭代极快，**会有破坏性变更**（官方原话）。

## 环境要求

- Node.js `^22.19.0 || >=24.0.0`（官方要求）
- pnpm（推荐 10+，官方仓库使用 11.x）
- 一个 DeepSeek API Key（对话和联网搜索需要）

检查环境：

```sh
node -v
pnpm -v
```

如果 `node` 找不到，去 https://nodejs.org 安装 LTS 或 24.x；Windows 安装后需要**新开终端**让 PATH 生效。

## 方式一：npm 一行命令启动（推荐新手）

```sh
npx @deepseek-ai/dsh web
```

实测输出（2026-08-15，dsh 0.1.0-rc.6）：

```text
dsh web: http://127.0.0.1:3080
```

浏览器打开 http://127.0.0.1:3080 即可。如果 3080 被占用：

```sh
npx @deepseek-ai/dsh web --port 4099
```

也可以用 `--host` 指定监听地址（默认 127.0.0.1）。

## 方式二：从源码运行

适合想读源码、改源码或给官方提反馈的开发者：

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

官方仓库是 pnpm workspace（vendor + packages/*/* + apps），`pnpm install` 需要几分钟，属正常现象。源码运行时 CLI 通过 tsx 直接跑 `apps/cli/src/bin.ts`。

> 注意：官方仓库根目录有 `AGENTS.md`，给官方贡献前请先读它（重要约定：注册都是 effect、构建前先跑针对性检查等）。

## 第一次对话：配置模型

Web UI 打开后：

1. 进入 **Providers / Models** 页面（左侧设置）。
2. 默认模型是 `deepseek-v4-flash`（provider 为 `deepseek-official`）。
3. 填入你的 DeepSeek API Key。也可以提前设好环境变量：

```sh
# PowerShell
$env:DEEPSEEK_API_KEY = "sk-你的key"
```

4. 回到会话页，输入你的第一句话，例如：

> 你好，介绍一下你自己，并列出你可以使用的工具。

没有 API Key 时，界面能打开但对话会报鉴权错误——这是预期行为，不是 bug。

## Web UI 速览

- **会话（Session）**：一次独立对话，状态（todo、计划、文件改动）按会话隔离。
- **工具卡片**：模型调用工具时，右侧/下方会显示卡片（终端、diff、搜索等），工具作者通过 `presentCall`/`presentResult` 声明卡片形态。
- **模型设置**：选择 provider/model、自定义模型地址（OpenAI 兼容接口也可配置）。
- **时区**：浏览器会把 IANA 时区附到每次请求，模型默认按你的时区理解"今天/明天"。

## 常见坑（实测踩过）

### 1. 端口 3080 被占用

```text
listen EADDRINUSE: address already in use 127.0.0.1:3080
```

解决：`--port 0` 让系统随机分配，或指定 `--port 4099`。

### 2. 没有 Node / pnpm 命令

Windows 上最常见的坑。安装 Node 后必须**新开终端**；pnpm 可用 `corepack enable` 启用，或 `npm install -g pnpm`。

### 3. `npx @deepseek-ai/dsh web` 提示版本错乱

社区插件请认准 `0.1.0-rc.6` 这条 release train。npm 上 `@deepseek-ai/dsh-tools` 的 latest 标签仍指向旧的 `0.0.1-rc.1`，它依赖未发布的包，**独立安装会 404 失败**。写插件时用 `^0.1.0-rc.6`。

### 4. 中文文档乱码

仓库部分中文文档在 Windows PowerShell 默认编码下显示乱码（UTF-8 被按 GBK 读取）。用 VS Code 打开或执行：

```powershell
Get-Content docs/xxx.md -Encoding UTF8
```

### 5. 插件装不上：git 安装需要允许构建

`dsh plugin add github:xxx/yyy` 时 pnpm 默认拒绝 git 依赖的 `prepare` 脚本。dsh 会提示你把一行 `allowBuilds:` 配置加进 profile 的 `pnpm-workspace.yaml`，照做即可；只对信得过的源码仓库授权。

## FAQ

**Q：dsh 需要 GPU 吗？**
不需要。dsh 是框架，模型走 DeepSeek API（也可配其他 OpenAI 兼容 provider）。

**Q：官方接受 PR 吗？**
目前不接受外部 PR（CONTRIBUTING.md 明确说明），但非常欢迎 Discussions 反馈、插件生态、教程和答疑。

**Q：`web` 和 `--profile web` 什么关系？**
`dsh web` 是 `dsh --profile web` 的别名。`dsh plugin --profile web add <包>` 可以把插件装进 web profile。

**Q：插件会弄坏我的会话数据吗？**
会话有持久化（默认 JSONL，可选 SQLite），插件加载失败会报错但不销毁已有会话；配置改动会热替换插件（HMR）。

下一篇：[《理解 dsh 架构：一切皆插件与 Cordis》](./zh-02-architecture.md)
