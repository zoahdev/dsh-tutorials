# dsh-plugin-doctor-action：把发布前体检装进 CI

> 作者：zoahdev 路 2026-08-16 路 仓库 https://github.com/zoahdev/dsh-plugin-doctor-action

## 一句话

插件"能加载 ≠ 能调用"（#1965/#1697/#2002）。这个 GitHub Action 把 dsh-plugin-doctor 的检查变成一行 CI 门禁，每个插件作者都能在用户安装前跑一遍维护者会跑的检查。

## 用法

```yaml
name: plugin
on: [push, pull_request]

jobs:
  doctor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: zoahdev/dsh-plugin-doctor-action@v1
        with:
          path: .
          # full: 'true'   # 追加 build + pack + 全新 profile 安装冒烟
```

## 它检查什么

- manifest 结构、patch 有效性、入口点（main/exports 是否指向真实文件）、files 白名单；
- 失败/警告直接变 GitHub annotations 和 run-summary，`ok` 输出可做 CI 门禁；
- 从固定 Release tarball 安装，不依赖 npm 发布。

## 为什么值得

dsh-plugin-template 的 CI 已经内置了这个 job——从模板 fork 出去的插件天然继承门禁。你的仓库也加一行，发布前就少一类"用户装不上"的 bug。
