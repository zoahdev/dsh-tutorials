# Sandbox Trust Boundaries and the Fail-Closed Principle

> For operators deploying dsh, plugin authors, and security reviewers. This chapter turns three 2026-08 community security threads (#3245/#3212/#3223) into an actionable checklist. All code references are against official main HEAD `99f6f02` (rc.7).

## 1. The one-line principle

**Trust boundaries must be enforceable, not declarable.** A README that says "this is isolated" is not isolation; a sandbox that says "read-only" must make every execution path actually read-only. A single bypass point invalidates the whole boundary.

## 2. Case study: #3245 run_code sandbox escape (Critical)

### What happened

Every normal dsh execution path honors the file-effect sandbox:
- bash → `bash-sandbox` calls `ctx.sandbox.confine(...)` (Landlock/bwrap/Seatbelt);
- filesystem tools → `fs-sandbox` fence (read-only refuses every mutation; workspace-write confines to the workspace root).

But **Code Mode's `run_code`** takes a third path: `code-runtime-worker-thread` spawns a Node worker with `new Worker(path, { env: {}, execArgv: [] })` and the model program runs under `new AsyncFunction(...)` — **no file-effect policy at all** (no `ctx.sandbox.confine`, no `--experimental-permission`). The worker shares the host process UID and kernel, so the program can `await import('node:fs')` to read/write any file and `await import('node:child_process')` to spawn unconfined subprocesses.

The default sandbox mode is read-only, so "Code Mode + default sandbox" = full escape, CVSS 10.0/9.8.

### Lesson 1: enumerate every execution path and check each against the boundary

When reviewing a sandbox, do not ask "is there a sandbox" — ask whether **every path that can execute code** goes through the same policy surface:

| Path | Sandboxed (rc.7 pre-fix)? |
|---|---|
| bash / pwsh | ✅ `ctx.sandbox.confine` |
| fs tools | ✅ fence + path canonicalization |
| run_code (worker-thread) | ❌ no file-effect policy |
| workflow worker-thread (node:vm) | ❌ same class of unconfined surface |
| cordis-host-runner dynamic plugins | ❌ documented as "not containment" |

### Lesson 2: fail-closed beats fail-open

Before the root fix (confining the worker via `ctx.sandbox.confine`) lands, the correct minimal mitigation is **fail-closed**: worker-thread isolation + confined policy (read-only/workspace-write) → refuse execution instead of silently running. Shipped branch `fix/run-code-failclosed-sandbox`: resolves the sandbox policy before dispatch; `runtime.isolation === 'worker-thread' && policyMode !== 'danger-full-access'` → throws (with three ways out: DSH_TOOLS_MODE=native / danger-full-access / a confinable runtime).

## 3. Case study: #3212 allow-lists over deny-lists

The principle from the dsh-crew discussion: **a textual safety belt is policy, not a security boundary**. Telling the agent "do not run dangerous commands" is not the same as the sandbox "not allowed" — the former can be bypassed (echo, command-line, MCP), the latter is kernel-enforced.

Audit your deployment: if a security guarantee lives only in a system prompt or plugin description, it is not a boundary; turn it into sandbox policy / permission enumeration / an approval gate.

## 4. Case study: #3223 attestation anchors (verifiability)

The remote-deployment trust boundary trilogy (#3209/#3210/#3211) is about **verifiable, not declarable**:
- externalUrl = say the right address, Tailscale-User-Login = say the right identity, surface = wire them together — none of them changes the trust fence;
- the ToolRuntime attestation/settlement vocabulary (#3223) should converge on CHA2A (#3192) instead of spawning a third dialect.

Operator self-check: is your "trusted source" **process-enforced** (signature verification, token validation) or **conventional** (docs say "only access from the intranet")? The latter is not a control in a threat model.

## 5. A reusable checklist

For any dsh security review:

1. **Enumerate execution surfaces**: bash / fs / run_code / workflow / cordis-runner / MCP — map each path to a policy surface, mark ✅/❌;
2. **Find bypasses**: for each ❌ ask "is it reachable under default config?" (#3245 = "Code Mode must be enabled, but enabling it makes the escape default-reachable");
3. **Fail closed**: when a root fix cannot land immediately, make "confined policy × unconfined path" **error** instead of running;
4. **Check doc honesty**: a "containment, not a security boundary" note must translate into a user-understandable trust level (worker-thread ≈ danger-full-access);
5. **Regression tests**: give every boundary an "escape attempt" case (the #3245 patch test = read-only + worker-thread must refuse).

## 6. Related community assets

- Family map family 13 "unconfined execution surfaces bypass the sandbox": dsh-ecosystem/docs/bug-families.md
- Patch queue #48 (#3245 fail-closed): dsh-docs/docs/specs/upstream-patches.md
- Tutorial zh-25: community contribution workflow (turning findings like this into evidence-grade replies)
- Tutorial zh-27: family-map triage (check the family before posting a new bug)
