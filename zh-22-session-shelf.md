# dsh-shelf：给你的 dsh 会话一个书架

> 作者：zoahdev 路 2026-08-15 路 仓库 https://github.com/zoahdev/dsh-shelf（v0.2.0）

## 一句话

每个 dsh 用户都会攒下几百个会话，但没人做生命周期管理（官方 #1990/#1991 都悬着）。dsh-shelf 就是那层书架：导出、归档、回收站、搜索、统计，零依赖、默认只读。

## 1. 常用命令

```sh
npx dsh-shelf list                          # 全部会话
npx dsh-shelf search "parser bug"           # 头 + 正文搜索
npx dsh-shelf export <id> --format md       # Markdown 对话记录
npx dsh-shelf archive <id> / restore <id>   # 归档 / 恢复
npx dsh-shelf trash <id> / restore-trash <id>
npx dsh-shelf report                        # 周报 digest
npx dsh-shelf archive-old 30 --yes          # 归档 30 天前的会话
```

## 2. 安全模型

- 列表/统计/导出/搜索**严格只读**；
- 归档/回收站是**移动**不是删除；引擎从不永久删除任何东西；
- `archive-old` 默认预演，`--yes` 才执行；
- Zstandard 压缩会话会被识别并提示（v0.1 明文导出，raw jsonl 保留）。

## 3. 为什么值得

会话是 agent 的记忆资产；记忆类插件很多，治理类没有。先占住 CLI 层（可脚本化、可进 CI），下一站是 Web UI、FTS5 中文搜索（对齐 #1999）、自动归档。

## 4. 相关

- 官方讨论：#1990（无法删除会话）、#1991（存档无法查看/恢复）
- awesome-dsh-plugin PR #652（Sessions & Messages）
