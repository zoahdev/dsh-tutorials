# dsh community contribution workflow: from bug reports to evidence-grade replies

> For anyone who wants to consistently publish high-quality contributions in the official dsh Discussions. Everything below comes from the 2026-08-18~19 campaign (7 evidence-grade replies + 1 triage script + 2 fix-day tools); commands tested on Windows / Node 24 / gh CLI.

## 1. Triage first: who is unanswered?

The official discussion board gets dozens of posts a day. Run the triage script instead of eyeballing:

```bash
node dsh-ecosystem/scripts/discussion-triage.mjs --since 24
```

It fetches the newest 100 posts with comment authors in ONE GraphQL request and prints `# / comments / ours(✅) / title`, then lists **0-comment candidates** at the end. Notes:

- `--since 24` (hours), `--all` (no window), `--json` (machine-readable)
- "ours" is decided by the current gh login (`gh api user --jq .login`); already-answered posts are flagged automatically
- Discipline: reply to each thread at most once; when the OP already has a full root cause, reply with **verification + increment only**

## 2. Verify before you reply: diff against current main
The last step can be automated: `node dsh-ecosystem/scripts/verify-citation.mjs <repo> <file:line> [--sha <HEAD>]` — validates the line exists, prints it, and compares HEAD, so you never cite a line you have not read.

For any post that cites source locations, verify first — three steps:

```bash
# 1) Shallow-clone the official repo (once)
git clone --depth 1 https://github.com/deepseek-ai/deepseek-harness.git

# 2) Record HEAD and state the verification baseline in your reply
git log -1 --format="%H %s"

# 3) Locate with rg, confirm line by line
rg -n "listArtifacts|spillAll|SIGINT" packages/session/session-persistence-jsonl/src packages/subprocess/subprocess-local/src apps/cli/src
```

Real campaign results:

| Thread | Verification |
|---|---|
| #675 SQLite torn-tail | OP's cited lines hold line-by-line on current main → added tornFrom's single consumer + regression-test suggestion |
| #1047 session.list 500 | Holds and unfixed rc.6→rc.7 → added two more unisolated throw sites in the same loop |
| #3190 spill ENOENT | Holds → pointed out the existing spillDisabled+discardSpill degrade pattern and gave a minimal patch |
| #3195 CTRL_C_EVENT | Holds and unfixed → added SIGBREAK same-channel risk + the Node-layer CREATE_NEW_PROCESS_GROUP constraint |

"Verified, still unfixed" is the most useful signal to maintainers: it says the report is credible and worth prioritizing, with an exact baseline.

## 3. Empirics: run the assertion locally when you can

Beyond source citations, reproduce what can be reproduced. Example: the CJK retrieval thread (#3206) claimed unicode61 is useless for Chinese and trigram hits; verified with Node 24's built-in node:sqlite:

```js
import { DatabaseSync } from 'node:sqlite'
const db = new DatabaseSync(':memory:')
db.exec("CREATE VIRTUAL TABLE t USING fts5(content, tokenize='unicode61')")
db.exec("CREATE VIRTUAL TABLE t2 USING fts5(content, tokenize='trigram')")
db.prepare('INSERT INTO t(content) VALUES (?)').run('索引优化减少Token消耗的句子')
db.prepare('INSERT INTO t2(content) VALUES (?)').run('索引优化减少Token消耗的句子')
console.log(db.prepare('SELECT count(*) c FROM t WHERE t MATCH ?').get('Token消耗').c)  // 0
console.log(db.prepare('SELECT count(*) c FROM t2 WHERE t2 MATCH ?').get('Token消耗').c) // 1
```

That also pinned an edge the OP's table did not show: **2-char CJK queries (e.g. "句子") return 0 on both lexical arms** (trigram needs >= 3 chars). A comment with measured data beats pure reasoning by an order of magnitude.

## 4. Reply via GraphQL, not REST

Creating discussion comments over REST 404s for ordinary tokens (GET works, POST does not). Use GraphQL:

```bash
# get the discussion node id
gh api repos/deepseek-ai/deepseek-harness/discussions/3174 --jq .node_id
```

```json
{
  "query": "mutation AddComment($discussionId: ID!, $body: String!) { addDiscussionComment(input: {discussionId: $discussionId, body: $body}) { comment { id url } } }",
  "variables": { "discussionId": "D_kwD...", "body": "Markdown body" }
}
```

```bash
# submit via `gh api graphql --input -` so shell escaping cannot mangle the body
```

## 5. Family-ize: merge same-root reports and help maintainers prioritize

Two real families from the campaign:

- **#3190 (spill-dir ENOENT crash) + #3203 (windows-acl sandbox temp dir cleaned)** = "external OS temp cleanup breaks a running dsh". Same root: long-lived per-process private %TEMP% dirs have no protection against external cleanup. Unified fix: recreate/degrade at the boundary where the dir is consumed.
- **#675 (one bad row cascades into deleting valid rows) + #1047 (one bad file wipes the whole list)** = "bad-artifact isolation". Recommended a shared regression-test convention: corrupt one artifact, assert the rest still work.

Linking sibling threads explicitly in replies ("#1047 and #675 are the same family") gives maintainers the full picture in one pass.

## 6. Anti-spam discipline

- Prefer 0-comment threads; do not repeat threads that already have solid replies
- When the OP ships a full fix proposal, reply as "verification + increment", not a rewrite
- Every reply contains only evidenced claims; mark speculation explicitly as unverified
- Reply in the thread's language; include line numbers / HEAD / test output so claims are re-checkable

## 7. Tooling inventory

| Tool | Where | Purpose |
|---|---|---|
| discussion-triage.mjs | dsh-ecosystem/scripts | nightly reply queue (one request, flags ours) |
| verify-2763-fix.mjs | dsh-ecosystem/scripts | auto re-scan + re-score + report on the #2763 fix day |
| full-registry-scan.mjs | dsh-ecosystem/scripts | full-registry supply-chain scan (325 plugins) |
| restore-publish.ps1 | local work/ | re-trigger the listing gate after repos age in (step 7/8) |

> The official awesome-dsh-plugin Submission gate re-runs "aged in" PRs automatically (regate.yml every 6h), so gates turn green even without manual action; an active re-trigger just makes it green at the earliest moment.