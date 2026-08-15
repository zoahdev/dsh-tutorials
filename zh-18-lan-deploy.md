# 内网部署 dsh web：非安全上下文崩溃的根因与修法

> 作者：zoahdev 路 2026-08-15 路 对应上游补丁 https://github.com/zoahdev/deepseek-harness/tree/fix/web-crypto-randomuuid-insecure-context

## 一句话

局域网用明文 HTTP 打开 dsh web，设置页报 `crypto.randomUUID is not a function`——因为该 API 只在安全上下文（HTTPS / localhost）里存在。修法不是换浏览器，而是给浏览器侧 UUID 生成加兜底。

## 1. 现象

```yaml
# profile patch 让 web 绑定 0.0.0.0
- id: webserver
  config:
    host: 0.0.0.0
    port: !!js ctx.webStartup.port ?? 3080
```

另一台设备打开 `http://<内网IP>:3080` → 设置 → 模型 → `加载提供方目录失败: crypto.randomUUID is not a function`。

## 2. 根因

浏览器全局 `crypto.randomUUID()` 只在安全上下文暴露。局域网 IP + 明文 HTTP 是非安全上下文，该值为 `undefined`；前端 4 处 id 生成（RPC id、附件 id、消息 id、实例 token）直接调用它，TypeError 在请求发出前就抛了。

## 3. 修复（上游补丁）

`fix/web-crypto-randomuuid-insecure-context`：

- 新增零依赖 `@deepseek-ai/dsh-random-uuid`：优先 `crypto.randomUUID()`，缺失时回退到 `crypto.getRandomValues()` 生成 RFC 4122 v4；
- 全部浏览器侧调用点改走它；打包产物里不再有直接 `crypto.randomUUID()` mint。

验证：`build:lib:host` + `build:lib:client` 全绿，定向 vitest 782/782。

## 4. 部署建议

- 受信内网 + 明文 HTTP：可用（补丁后不崩溃），但传输仍无加密；
- 公网/半受信网络：请在前面放 HTTPS（反向代理或自签证书），并限制网络可达；
- 需要远程访问时，用 Tailscale/WireGuard 这类加密通道，比裸开端口安全得多。

## 5. 相关

- 官方讨论：https://github.com/deepseek-ai/deepseek-harness/discussions/1919
- 社区插件兜底：dsh-web-lan-access、dsh-mobile-gate（已在 dsh-subscribe 注册表收录）
