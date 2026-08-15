# Efficient Codebase Retrieval: Make AGENTS.md a "Tools-First" Contract

> By zoahdev · 2026-08-15 · Aligns with official efficiency feedback discussion #1864

## Problem

A real session: one PowerShell recursive directory listing (`Get-ChildItem -Recurse -Depth 2 -Include *.md`) walked into node_modules symlinks, took 120s (timeout), and produced ~78KB of garbage output; the whole 9-step turn accumulated 307K input tokens.

Local micro-benchmark: the same command on a clean tree takes 4ms — the hang is not the command itself but **symlink/junction cycles, network drives, or huge trees**. The fix is not "list faster", it's "don't use shell recursion by default".

## Contract: prefer built-in retrieval tools

Add this to your repo's `AGENTS.md` (or dsh global rules) so every retrieval follows the same cheap path:

```markdown
## Retrieval contract

1. Prefer Glob/search tools for file discovery; never `find` / `Get-ChildItem -Recurse` / `tree`.
2. Search tools must exclude `node_modules`, `.git`, `dist`, `.next`, `__pycache__`.
3. Read files with offset/limit, not whole-file; grep with head_limit + minimal context.
4. Batch-read related files in parallel to cut turns.
5. Intermediate steps report conclusions only; final answer matches the question's scope.
6. Keep the prompt prefix stable to keep hitting the context cache.
```

## Why it works

- Built-in Glob/Grep/Read are runtime-implemented: dependency trees skipped, output truncated — no 78KB garbage into context.
- Parallel batch reads compress 9 steps into 4-5, cutting cumulative re-sent input.
- Output limits directly reduce the "recomputed" fraction (~49K of the observed 307K input was garbage/whole-file reads).

## Guardrail suggestions (for upstream)

- Add a guardrail or hint for shell "recursive listing" commands that hit node_modules symlinks.
- Default tool-output caps (bytes/lines) with truncation and a visible marker.
- Ship the Retrieval contract as a built-in AGENTS.md template in official presets.

## Related

- Official feedback: https://github.com/deepseek-ai/deepseek-harness/discussions/1864
