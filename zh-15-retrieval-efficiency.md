# 高效代码库检索：把 AGENTS.md 写成"工具优先"约定

> 作者：zoahdev · 2026-08-15 · 对应官方效率反馈 discussion #1864

## 问题

真实会话观测：一次 PowerShell 递归列目录（`Get-ChildItem -Recurse -Depth 2 -Include *.md`）遍历进 node_modules 的符号链接，单独耗时 120 秒超时，产出约 78KB 垃圾输出；整轮 9 步累计输入 307K token。

本地微基准：干净目录树上同样的命令只需 4ms——卡死不是命令本身，而是**符号链接/junction 环、网络盘或超大目录**。所以解决方式不是"更快地列目录"，而是**默认不要用 shell 递归列目录**。

## 约定：检索优先用内置工具

把下面这段写进仓库的 `AGENTS.md`（或 dsh 全局规则），让 agent 每次检索都走同一套低成本路径：

```markdown
## Retrieval contract（检索约定）

1. 文件发现优先用 Glob/搜索工具，不用 `find`/`Get-ChildItem -Recurse`/`tree`。
2. 搜索工具必须排除 `node_modules`、`.git`、`dist`、`.next`、`__pycache__`。
3. 读取文件用 offset/limit，不整篇读；grep 输出用 head_limit + 少量上下文行。
4. 相关文件并行批读取，减少轮次。
5. 中间步骤只输出结论，最终答案控制在与问题匹配的长度。
6. 提示词前缀保持稳定，持续命中上下文缓存。
```

## 为什么有效

- 内置 Glob/Grep/Read 由运行时实现，天然跳过依赖树、支持截断——不会把 78KB 垃圾灌进上下文；
- 并行批读把 9 步压到 4-5 步，减少每步重发全部上下文的累计输入；
- 输出限流直接砍掉"重新计算"部分（观测中 307K 输入里约 49K 是垃圾/整篇读造成的）。

## 护栏建议（写给官方）

- 工具调用侧对"递归列目录"类 shell 命令加护栏或提示，命中 node_modules 符号链接时警告；
- 工具输出默认上限（字节/行数），超出截断并在呈现时标明；
- 把上面的 Retrieval contract 作为模板内置到官方 presets 的 AGENTS.md。

## 相关

- 官方反馈：https://github.com/deepseek-ai/deepseek-harness/discussions/1864
