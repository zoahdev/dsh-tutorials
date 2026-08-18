# Passing the awesome-dsh-plugin Submission gate (complete guide)

> For anyone submitting a dsh plugin to the [awesome-dsh-plugin](https://github.com/awesome-dsh-plugin/awesome-dsh-plugin) curated list. Every behavior below was measured in practice (2026-08-19, repo-age gate scenario).

## 1. What the gate checks

The `Submission gate` on listing PRs checks three things (`scripts/check-submission.mjs` + `pr-gate.yml`):

1. **A `dsh.bundle` manifest exists** — your repo/package must declare the bundle, otherwise it is rejected outright;
2. **The repo is at least 1 day old** (from the repo `created_at`, NOT the PR creation time);
3. **The repo has >= 10 commits**.

All three pass -> green `success`; any failure -> red `failure`, and the PR shows `mergeStateStatus: UNSTABLE` (mergeable but checks failing).

## 2. The most common red light: repo age

New repos almost always hit `repository is X.X days old (needs 1) — resubmit in about 15h`. This is intentional (it stops one-shot accounts from flooding the list), not a problem with your submission.

## 3. How it turns green (the key mechanism)

The gate verdict is a snapshot: it does not refresh itself once the age passes; it needs a new push/check. Two paths:

**A. Official fallback (do nothing)**

The repo's `regate.yml` runs every 6h (cron at :19 UTC) and automatically re-runs PRs that "aged in" (failure reason matching `days old|commit(s)`). After the age passes, the gate turns green within 6h automatically. **All you have to do is wait.**

**B. Manual early trigger (when you want it green ASAP)**

Push an empty commit to the PR branch to re-fire pr-check -> gate re-evaluation:

```bash
git checkout <pr-branch>
git commit --allow-empty -m "chore: gate re-trigger stamp"
git push origin <pr-branch>
```

⚠️ **Do not try to re-run the workflow via API**: `POST /actions/runs/{id}/rerun` returns `403 Must have admin rights to Repository` (only org admins can rerun). Pushing from your fork is the working path.

## 4. Other common red lights

| Symptom | Cause | Fix |
|---|---|---|
| `dsh.bundle` not declared | manifest missing `dsh.bundle` | add the bundle/patch declaration |
| `repository is 0.1 days old` | age gate | wait 1 day (regate handles it) |
| `commit(s)` insufficient | repo has < 10 commits | add real commits (README/SECURITY/CHANGELOG/CONTRIBUTING all count); do NOT pad with empty commits |
| `check` (PR check) fails | entry format/content issue | read that check's output and fix; the gate does not re-run content-broken PRs (regate only handles age/commit classes) |

## 5. Our measured data

- 8 repos (10 commits each) all passed the commit check first try; the only red light was age;
- Eligibility time = `repo created_at + 24h`, precise to the minute (GitHub API `created_at`);
- After a manual stamp push, the gate turns green once pr-check completes; the official regate also covers you within the 6h window.

## 6. Check your eligibility time

```bash
gh api repos/<owner>/<repo> --jq .created_at   # repo creation time
# eligible = created_at + 24h (UTC)
```

After listing, update your README badges/status and share progress in an official Discussions showcase thread (like the #3123 pattern).