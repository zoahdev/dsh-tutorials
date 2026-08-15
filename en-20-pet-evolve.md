# dsh-pet-evolve: turn your agent's growth into a pet

> By zoahdev 路 2026-08-15 路 Repo: https://github.com/zoahdev/dsh-pet-evolve (v0.2.0)

## TL;DR

Pet galleries show you sprites; every form here is **earned from real agent signals** - verified rules, completed sessions, tool calls, and compactions all convert to XP.

## 1. Evolution

Egg → Baby → Teen → Adult → Legend (300 / 800 / 1600 / 3000 XP). One level per 100 XP. The pet is drawn live on canvas with four skins: whale / cat / robot / ghost.

## 2. Real signals

```sh
npx dsh-pet-evolve --profile ~/.dsh/profiles/web
```

- `rule_verified`: verified rules in the rule library (dsh-rule-evolve compatible)
- `session_completed`: finished sessions
- `tool_call`: tool activity in session logs
- `compaction`: compaction summaries

The pet mirrors agent state: bounces with a gear while working, confetti on done, sad on failed, blinks while idle.

## 3. Share

One click exports a 1200x630 growth card PNG: "My agent pet reached Legend - Level 12 - 25 rules verified".

## 4. DSH plugin mode

```sh
dsh plugin --profile web add github:zoahdev/dsh-pet-evolve
```

Committed `lib/` means no build step at install (the #1965 lesson, applied).

## 5. Privacy

Reads only the local files you point it at; zero telemetry; the share card is generated in your browser.
