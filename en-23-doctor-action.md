# dsh-plugin-doctor-action: pre-publish checks in CI

> By zoahdev 路 2026-08-16 路 Repo: https://github.com/zoahdev/dsh-plugin-doctor-action

## TL;DR

Plugins can "load" yet be uncallable (#1965/#1697/#2002). This GitHub Action turns dsh-plugin-doctor's checks into a one-line CI gate so every plugin author runs the maintainer's checks before users install.

## Usage

```yaml
name: plugin
on: [push, pull_request]

jobs:
  doctor:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: zoahdev/dsh-plugin-doctor-action@v1
        with:
          path: .
          # full: 'true'   # add build + pack + fresh-profile install smoke
```

## What it checks

- Manifest structure, patch validity, entry points (main/exports point at real files), files allowlist.
- Failures/warnings become GitHub annotations and the run summary; the `ok` output is CI-friendly.
- Installs from a pinned Release tarball, no npm dependency.

## Why

dsh-plugin-template already ships this job, so plugins forked from the template inherit the gate. Add one line to your repo and remove a whole class of "users can't install it" bugs before release.
