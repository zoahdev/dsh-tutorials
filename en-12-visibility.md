# Getting Noticed in the dsh Ecosystem: a 24-Hour Retrospective

> By zoahdev · 2026-08-15 · Everything here can be verified against real commits and this tutorial series

## The honest summary

There is no shortcut to fame in an early open-source ecosystem, but there is a repeatable path: build what the official discussions explicitly ask for → prove it with real runtime evidence → answer reviews with evidence, not arguments → cross-link with tools in the same niche → turn knowledge into tutorials. In 24 hours that produced: 5 repos, 195+ tools, 23 tutorial pages, 10+ official discussion replies, and 2 pending listing PRs.

Honest disclosure: stars are still 0. The channels are set; external review and traffic are pending. This article is about maximizing the chance of being seen — not about gaming metrics.

## Five moves, in order

### 1. Build what the official discussions explicitly ask for

- RFC #1629 asked for `dsh plugin check` → dsh-plugin-doctor.
- #1715 asked "where is dsh-plugin-search?" → dsh-plugin-search.
- #1719 proposed `dsh doctor` → our `--env` implements it.

A named request is proof of demand; you only need to prove you shipped it and it works.

### 2. Smoke-test every feature against real APIs

"It loads" is not "it works". Every tool runs against the real API, which caught a string of documented-but-broken endpoints:

- Bitbucket Cloud: 404/410 anonymous → not shipped;
- Maven Central: repeated timeouts → not shipped;
- Hugging Face `/api/tags`: 401 → not shipped;
- Stack Overflow requires `site=stackoverflow` (old tools 400'd) → fixed;
- PR-disabled repos broke the weekly digest → per-surface 404 tolerance.

The failures are trust assets — more convincing than any "stable and reliable" claim.

### 3. Answer reviews with evidence

The awesome-dsh-plugin maintainer closed our PR #352 because a pure CLI wasn't listable. Instead of arguing: added the plugin shell (`dsh.bundle` + `cordis.patch.yml` + a `plugin_check` model tool), proved every item with CI, and asked for a re-review with the full verification output attached.

Later, the same maintainer independently verified our tool in another thread and confirmed it works — that is the long-term return on evidence.

### 4. Cross-link with tools in the same niche

Our doctor complements two other community diagnostics (moonquake2004/dsh-doctor and the dsh-diagnose skill): prevention / probing / symptom understanding. We link each other in READMEs and agreed on one JSON schema. An ecosystem *organizer* is remembered longer than an isolated tool.

### 5. Turn knowledge into tutorials

23 bilingual pages, every command actually run: getting started, plugin development, pre-publish checks, peer-version traps, the awesome-list review checklist, marketplace publishing, and debugging the `undefined.prepare` crash family. Tutorials are compound interest — they get read while you sleep.

## Checklist for newcomers

```text
[ ] Find a demand named in official discussions that you can execute well
[ ] Ship an installable artifact (npm / github:owner/repo / tarball)
[ ] Smoke-test every feature against real APIs; document the failures
[ ] CI proves: pack → fresh profile install → real tool invocation → web boot
[ ] Submit to listings (awesome list / marketplace); answer reviews with evidence
[ ] Cross-link with tools in the same niche
[ ] Write 2-3 bilingual tutorials
[ ] Honestly record what you could not verify
```

## Resources

- Integration: [dsh-github-intelligence](https://github.com/zoahdev/dsh-github-intelligence)
- Doctor: [dsh-plugin-doctor](https://github.com/zoahdev/dsh-plugin-doctor)
- Search: [dsh-plugin-search](https://github.com/zoahdev/dsh-plugin-search)
- Template: [dsh-plugin-template](https://github.com/zoahdev/dsh-plugin-template)
- Tutorials: https://zoahdev.github.io/dsh-tutorials/
