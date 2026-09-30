---
name: gx-skills
description: "Use when working with gx (galaxy-flow) — the GXL workflow engine and CLI: running workflows (gx run / gx adm), initializing projects, authoring GXL with built-in gx.* capabilities, and troubleshooting gx execution. Routes to the gx-engineering skill."
---

# Gx Skills

Top-level collection for `gx` (galaxy-flow) skills.

## Routing

- For `gx` — `gx run/adm/init/mod/doc/check/self`, GXL workflow authoring and pitfalls (`gx.shell`/`gx.cmd`, `silence`, backgrounding), built-in `gx.*` capabilities, and `_gal/` conventions — read `skills/gx-engineering/SKILL.md`.
- If the nested skill references files, resolve them relative to its own directory.
- Do not load every nested skill by default. Pick only the one matching the user's task.

## Workspace Assumptions

- `galaxy-flow` (gx) source: https://github.com/galaxio-labs/galaxy-flow
- Related collection: `gops` (galaxy-ops) skills live in https://github.com/galaxio-labs/gops-skills
- Prefer source and tests over stale docs when they disagree.

## Validation

- gx: `cargo build` / `cargo test` in `galaxy-flow`.
