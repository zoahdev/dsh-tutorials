# Let Agents Evolve: dsh-rule-evolve in Practice

> By zoahdev · 2026-08-15 · Repo: https://github.com/zoahdev/dsh-rule-evolve (v0.9.0)

## TL;DR

Turn "failure → rule → verify → apply → recall → retire" into the agent's own loop: every lesson becomes a rule checked by a real verification, installed into the profile's AGENTS.md, and active in the next session.

## 1. Install

```sh
dsh plugin --profile web add github:zoahdev/dsh-rule-evolve#path:/plugin
```

Four tools: `evolve_learn`, `evolve_apply` (refuses unverified installs), `evolve_touch` (usage), `evolve_recall` (local retrieval).

## 2. Full CLI loop

```sh
node scripts/dsh-evolve.mjs extract --task "publish to npm" --from install.log --out experience.jsonl
node scripts/dsh-evolve.mjs verify --experience experience.jsonl --dir ./my-plugin
node scripts/dsh-evolve.mjs evolve --experience experience.jsonl --dir ./my-plugin --profile web
node scripts/dsh-evolve.mjs recall --experience experience.jsonl --query "windows bash"
node scripts/dsh-evolve.mjs score / prune / merge-duplicates
```

## 3. Real evidence (dogfood)

- 6 rules extracted from a real npm ENEEDAUTH failure log in one command
- `tool-verify` on our doctor repo: 8/8 PASS → READY ✅ (before v1.10.1 the same gate caught its own lint bug)
- rule library: 5/5 active, 0 duplicates; "windows bash" recalls the #1856 rule first
- six cherry-pick-ready patches, all verified against the official test suite

## 4. Why this is self-evolution

Rules carry source, verification history, usage history, and lifecycle state. High-score rules survive; low-score rules are retired auditably — the library remembers what works, not everything.

## Related

- Discussion: https://github.com/deepseek-ai/deepseek-harness/discussions/1906
- Memory RFC: #1881 (dsh-rule-evolve is the production + validation half of Layer 1)
