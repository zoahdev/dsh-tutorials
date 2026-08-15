# One-Minute Due Diligence on Any GitHub Repo: dsh-github-intelligence in Action

> By zoahdev · 2026-08-15 · All output below is real GitHub API data

## Table of Contents

- [Install and go](#install-and-go)
- [Three prompts you will actually use](#three-prompts-you-will-actually-use)
- [What real output looks like](#what-real-output-looks-like)
- [The seven tools](#the-seven-tools)
- [Why it is rate-limit friendly](#why-it-is-rate-limit-friendly)
- [Pitfall: the issues endpoint mixes in pull requests](#pitfall-the-issues-endpoint-mixes-in-pull-requests)
- [Next steps](#next-steps)

## Install and go

```sh
dsh plugin --profile web add github:zoahdev/dsh-github-intelligence
```

Restart `dsh web` and just ask. No API key needed (60 requests/hour anonymous; the plugin's 60s TTL cache makes that budget go far).

## Three prompts you will actually use

1. "Give me a full report on `deepseek-ai/deepseek-harness`."
2. "What open issues does `ollama/ollama` have right now?"
3. "Compare the top 5 TypeScript agent frameworks by stars."

## What real output looks like

This is the real `github_repo_report` output for `deepseek-ai/deepseek-harness` on 2026-08-15:

```text
# deepseek-ai/deepseek-harness
deepseek-ai/deepseek-harness — DeepSeek Harness: Everything is a Plugin.
Stars: 98472 · Forks: 9214 · Open issues: 0 · Default branch: master
Language: TypeScript · License: MIT · Pushed: 2026-08-13
https://github.com/deepseek-ai/deepseek-harness

Latest release: none
Open issues: 0

Recent commits:
- 47f9438 Merge pull request #2519 from deepseek-harness/feat/npm-public (imccyu)
- abe560f release(dsh): 0.1.0-rc.5 (imccyu)
- 8c1e8d9 build(release): publish the dsh family publicly (imccyu)
- f26a6f6 Merge pull request #2520 from deepseek-harness/docs/paper (Tianyi Cui)
- 124aa5f Merge pull request #2521 from deepseek-harness/release/dsh-0.1.0-rc.3 (imccyu)

Top contributors: 1. tianyicui (5235) · 2. LegGasai (1361) · 3. imccyu (1168) · 4. Chinesezjc (587) · 5. turtle1999 (585)
```

One question = full overview + latest release + open issues + recent commits + top contributors. That is due diligence.

## The seven tools

| Tool | What it does |
|---|---|
| `github_repo` | Full overview (topics, default branch, archived status) |
| `github_releases` | Recent releases with body previews |
| `github_issues` | Issues by state |
| `github_pulls` | Pull requests by state (merge status included) |
| `github_contributors` | Top contributors by commit count |
| `github_search` | Repository search by stars or last update |
| `github_repo_report` | All of the above in one canonical answer |

## Why it is rate-limit friendly

The anonymous GitHub API allows 60 requests/hour. Every endpoint in this plugin has a 60s TTL cache: ask for the same repo report three times in a row and only the first costs 4 requests. The deep report runs its sub-calls in parallel and reuses the cache instead of stacking cost.

## Pitfall: the issues endpoint mixes in pull requests

GitHub's `/issues` endpoint returns pull requests by default. The plugin filters them out by the `pull_request` field in `listIssues` — this is the difference between "integrating" and "calling an API": same surface, clean data.

## Next steps

- Source and verification: [zoahdev/dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- More tutorials: [index](./README.md)
