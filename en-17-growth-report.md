# Make Agent Growth Visible: the Agent Growth Report

> By zoahdev · 2026-08-15 · Repo: https://github.com/zoahdev/dsh-rule-evolve (v0.10.2)

## TL;DR

One command turns the rule library and evolution history into a beautiful growth report (HTML + downloadable PNG share card): "My agent learned N rules, all verified by real checks."

## 1. Generate

```sh
node scripts/dsh-evolve.mjs dash --experience experience.jsonl --evolution EVOLUTION.md --out agent-growth.html
```

Self-contained HTML (zero deps, offline, bilingual): growth curve, library health, learned topics, most-used rules, evolution timeline, share card + PNG export.

## 2. In-session version

With the plugin installed, the agent reports its own growth:

```text
evolve_report(profile: "web")
→ "🐋 My DeepSeek Harness agent learned 11 rule(s), 11/11 verified by real checks, used 3 time(s)…"
```

## 3. Live example

https://github.com/zoahdev/dsh-rule-evolve/blob/main/examples/marathon/agent-growth.html

## 4. Why

Self-evolution only counts when it's visible. The growth report turns verification-driven learning into something shareable — a screenshot is worth a thousand commits.
