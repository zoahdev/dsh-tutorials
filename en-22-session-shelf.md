# dsh-shelf: give your dsh sessions a shelf

> By zoahdev 路 2026-08-15 路 Repo: https://github.com/zoahdev/dsh-shelf (v0.2.0)

## TL;DR

Every dsh user accumulates hundreds of sessions, yet no one builds lifecycle tooling (official #1990/#1991 are unanswered). dsh-shelf is that missing shelf: export, archive, trash, search, stats - zero dependencies, read-only by default.

## 1. Commands

```sh
npx dsh-shelf list
npx dsh-shelf search "parser bug"
npx dsh-shelf export <id> --format md
npx dsh-shelf archive <id> / restore <id>
npx dsh-shelf trash <id> / restore-trash <id>
npx dsh-shelf report
npx dsh-shelf archive-old 30 --yes
```

## 2. Safety model

- List/stats/export/search are **strictly read-only**.
- Archive/trash are **moves**, never deletes; the engine never permanently removes anything.
- `archive-old` is a dry run by default; `--yes` executes.
- Zstandard-compressed sessions are detected and reported.

## 3. Why

Sessions are an agent's memory assets; memory plugins are plentiful, governance is not. Own the CLI layer first (scriptable, CI-friendly), then Web UI, FTS5 Chinese search (#1999), and auto-archive.

## 4. References

- Official: #1990 (no way to delete a session), #1991 (archived sessions cannot be viewed/restored)
- awesome-dsh-plugin PR #652 (Sessions & Messages)
