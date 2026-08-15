# Deploying dsh web on a LAN: the insecure-context crash, root cause, and fix

> By zoahdev 路 2026-08-15 路 Upstream patch: https://github.com/zoahdev/deepseek-harness/tree/fix/web-crypto-randomuuid-insecure-context

## TL;DR

Opening dsh web over plain HTTP on a LAN IP throws `crypto.randomUUID is not a function` in Settings -> Models, because that API only exists in secure contexts (HTTPS / localhost). The fix is a fallback UUID generator in the browser bundle - not a browser change.

## 1. Symptom

```yaml
# profile patch to bind the web server to all interfaces
- id: webserver
  config:
    host: 0.0.0.0
    port: !!js ctx.webStartup.port ?? 3080
```

Open `http://<LAN-IP>:3080` from another device, go to Settings -> Models: `failed to load provider directory: crypto.randomUUID is not a function`.

## 2. Root cause

The browser global `crypto.randomUUID()` is only exposed in secure contexts. A LAN IP over plain HTTP is not secure, so the value is `undefined`; four browser-side id mints (RPC ids, attachment ids, message ids, instance token) call it directly and throw before any request is sent.

## 3. Fix (upstream patch)

`fix/web-crypto-randomuuid-insecure-context`:

- New zero-dependency `@deepseek-ai/dsh-random-uuid`: prefers `crypto.randomUUID()`, falls back to `crypto.getRandomValues()` for RFC 4122 v4 UUIDs.
- All browser-facing mints route through it; the built bundles contain no direct `crypto.randomUUID()` mint.

Verified: `build:lib:host` + `build:lib:client` green, targeted vitest 782/782.

## 4. Deployment guidance

- Trusted LAN over plain HTTP: works after the fix, but the transport is still unencrypted.
- Public or semi-trusted networks: terminate HTTPS in front (reverse proxy or self-signed cert) and restrict reachability.
- For remote access prefer an encrypted tunnel (Tailscale / WireGuard) over exposing a raw port.

## 5. References

- Official discussion: https://github.com/deepseek-ai/deepseek-harness/discussions/1919
- Community polyfill plugins: dsh-web-lan-access, dsh-mobile-gate (listed in the dsh-subscribe registry)
