# Bug-Family Triage: Classify and Verify Issues with the Family Map

> For anyone contributing to the dsh community: before answering a new bug report, check the family map first. The map and workflow below come from real operations on 2026-08-19 (30+ evidence-grade verification replies).

## 1. Why family-ize

In the dsh discussions, **the same root cause routinely shows up across many posts**: different environments, different symptoms, reporters who do not know about each other. Three costs:

- maintainers must piece threads together to see the real blast radius;
- every new thread re-explains everything from zero;
- a fix lands for "this thread" without closing "this class".

Family-izing groups threads by root cause: a new post is checked against the map first, and when it hits a family you cite the canonical thread and add incremental evidence instead of opening a parallel universe.

## 2. What the family map is

[The known bug family map](https://github.com/zoahdev/dsh-ecosystem/blob/main/docs/bug-families.md) (docs/bug-families.md in dsh-ecosystem) currently tracks 12 verified families. Each entry carries:

| Field | Meaning |
|---|---|
| Trigger model | How you get hit |
| Canonical thread(s) | The family's authoritative post numbers |
| Verified root cause | Official source file:line, directly checkable |
| Fix suggestion | Minimal patch + regression-test convention |
| Status | Whether upstream has fixed it |

The 12 families at a glance:

1. **npm dist-tag publishing** (#2763: latest stuck at 0.0.1-rc.1; 160/325 registry plugins affected)
2. **%TEMP% cleanup** (#1961/#3190/#3203: runtime-private dirs deleted → crash or fail-closed)
3. **Corrupt-artifact isolation** (#675/#1047/#3173: one bad file takes down lists/boot)
4. **Module double-instance symbol split** (#1697/#2660/#3033: the undefined.prepare family)
5. **Reasoning-field aliases** (#199: vLLM sends delta.reasoning; adapter reads only reasoning_content)
6. **Windows token/ACL sandbox** (#3207/#3216/#3195)
7. **Remote-deployment trust fences** (#3209/#3210/#3211)
8. **Plugin load/rollback** (#3173/#3213)
9. **Search/rendering/metadata** (#3202/#3206/#3177/#3111)
10. **Tool-result observability** (#3182: ok field unreliable; archive ≠ delete)
11. **Out-of-tree session-event envelope** (#3191/#1538/#1584/#1619/#2778: ignorable write-side gap; read side ready; implementation verified)
12. **Same-mode sandbox escalation false errors** (#3219: full-access requests full-access and gets "not strictly wider"; fix branch ready)

## 3. When you hit a family

1. **Cite the canonical thread** instead of re-analyzing — "this shares the #2660 root cause (double-instance symbol split)";
2. **Add incremental evidence**: new environment (macOS/WSL/Windows 10 vs 11), new trigger path, new field name, new measurement;
3. Provide a **minimal repro for your environment** (command + expected/actual) so maintainers can verify directly;
4. Only suggest a fix if you have a **new angle**; a plain "repro +1, env: …" is enough for a me-too.

Example (#3033): the reporter hit `undefined.prepare` on macOS after installing dsh-computer-use; argszero linked it into the #2660 family and sharpened the trigger model to "any second physical copy". That is the standard family contribution: one increment, whole family benefits.

## 4. When nothing matches

First run the [zh-25](./en-25-community-workflow.md) verification triple: shallow clone, pin HEAD, check citations.

```bash
git clone --depth 1 https://github.com/deepseek-ai/deepseek-harness work/src   # shallow clone
git log -1 --format="%H"                                                       # record the baseline
node work/dsh-ecosystem/scripts/verify-citation.mjs <repo> <file:line>         # line-number check
```

A confirmed new root cause is **a new family**: post the file:line evidence in your reply, then add an entry to the map (instructions at the top of the file). The map is a living asset, kept fresh by community backfill.

## 5. Family-specific test conventions

Do not paper over a family with one isolated test; each family has a canonical regression shape:

| Family | Test convention |
|---|---|
| Corrupt-artifact isolation | `[valid, bad, valid]` flow: bad records must not swallow valid ones or overwrite the latest |
| %TEMP% cleanup | delete the runtime dir → next call recreates (provider side); runner boundary stays fail-loud |
| Symbol split | double-copy load → same scheduler instance; orphaned tool_calls no longer replay into 400 |
| Reasoning aliases | three tests: alias accepted / official field wins / reasoning-only stop is NOT EMPTY_RESPONSE |
| Windows tokens | curl.exe regression (TLS handshake inside sandbox) + event-log ACL audit |

## 6. Our worked examples

- **#199 → family 5**: two reporters (DiGuStudent / dietmarscharf) filed the vLLM reasoning-field drop independently. After verifying `translate.ts:132`, we completed the mechanical chain "EMPTY_RESPONSE → retry-policy storm" and suggested merging both branches into one PR — two threads became one maintainer-ready PR description.
- **#3182 → families 9/10**: of the five findings, archive≠delete, the unreliable ok field, and node_modules scanning mapped to "tool-result observability" and "search semantics"; each was confirmed with file:line before replying.
- **#2011 → new family outside the map**: host.describe hardcoding 0.0.1 (api-proxy.ts:2863-2868) is a fresh root cause; the reply gives the contract mismatch and fix path, ready to backfill.

## 7. TL;DR

**Check the map first; when it hits, append increments; when it misses, found a new family.** Community contribution is not about reply count — it is about moving every new thread toward "maintainers can fix the whole class in one pass".
