# Understanding dsh Architecture — English Summary

Full Chinese version: [zh-02-architecture.md](./zh-02-architecture.md)

## The one-liner

> dsh = Cordis (plugin runtime) + capability packages (tools/llm/fs/shell/web/…) + an event-sourced session layer + a composition mechanism (profile/patch)

## Plugin forms

A plugin is a module exporting `name` and `apply(ctx)`. Three forms: function (default), object, or a class extending `Service` (when providing a service to other plugins).

## Core concepts

- **Context (`ctx`)**: the registration surface passed to `apply`. `ctx.on`, `ctx.effect`, `ctx.inject`, and service registries like `ctx.tools`.
- **`inject`**: declares required services; Cordis guarantees they are ready before `apply` runs.
- **`ctx.effect`**: registers a side effect with a cleanup function; unloads automatically. "Registrations are effects" is an official convention.
- **`defineTool`**: author-facing DSL for model-facing tools — validated parameters, canonical JSON output schema, pure `render`, optional `presentCall`/`presentResult` cards, and `exec.signal` cancellation.
- **Schemastery `Config`**: schema-validated configuration with defaults; no hardcoded tunables.

## Composition: bundle / profile / patch

- **bundle**: an npm package with a `dsh.bundle` manifest contributing a `cordis.patch.yml`.
- **profile**: an ordered, bootable composition under `$DSH_HOME/profiles/<name>`.
- **patch**: YAML patch rows; later layers win by `id`.

Order: profile bundles → profile patch → `$DSH_HOME` patch → CLI `--patch`.

## Repo layout (top-level)

`vendor/` (vendored Cordis), `packages/` (`@deepseek-ai/dsh-*` workspaces: core, api, llm, fs, shell, web, todo, plan, goal, skill, workflow, bundle…), `examples/` (runnable cordis.yml leaves), `docs/`, `website/`.

## dsh vs MCP

Native tools are registered directly via `defineTool`; MCP tools arrive through `@deepseek-ai/dsh-mcp-client` and are mapped into the same registry. They are complementary: MCP reuses existing ecosystems, native plugins allow deep customization (cards, policy, session state).

## Glossary

| Term | Meaning |
|---|---|
| harness | the framework layer an agent runs on |
| Context | Cordis registration surface |
| inject | declared service dependencies |
| effect | side effect with cleanup |
| canonical value | the tool's JSON result value |
| render intent | UI card kind (generic/terminal/diff/search/web) |
| bundle / profile / patch | distribution unit / bootable composition / overlay |
