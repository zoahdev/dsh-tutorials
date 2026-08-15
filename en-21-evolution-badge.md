# The evolution badge: your rule library as a README SVG

> By zoahdev 路 2026-08-15 路 dsh-rule-evolve v0.11.0

## TL;DR

`dsh-evolve badge` turns "how many verified rules did my agent learn" into an SVG badge you can embed in any README.

## 1. Usage

```sh
dsh-evolve badge --experience experience.jsonl --evolution EVOLUTION.md --out badge.svg
```

```markdown
![agent rules](badge.svg)
```

## 2. Tiers

By verified-rule count: `starting` (0) -> `learning` (1-4) -> `building` (5-9) -> `growing` (10-19) -> `legend` (20+), with matching colors.

## 3. CI-friendly

`--json` returns metadata plus the SVG payload, so release pipelines can write the badge straight into the README:

```sh
dsh-evolve badge --experience experience.jsonl --json > badge.json
```

## 4. Why

Self-evolution only counts when it is visible. The growth report is the long read, the share card is the image, the badge is the **always-on storefront** - every visitor sees your agent growing.
