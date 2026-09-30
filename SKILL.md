---
name: gx-skills
description: "Use when working with gx (galaxy-flow) — the GXL workflow engine and CLI: running workflows (gx run / gx adm), initializing projects, authoring GXL with built-in gx.* capabilities, and troubleshooting gx execution. Routes to the gx-engineering skill."
---

# Gx Skills

Top-level collection for `gx` (galaxy-flow) skills.

## Routing

- For `gx` — `gx run/adm/init/mod/doc/check/self/skill`, GXL authoring (structures, flow heads, annotations, control flow, variables, built-in constants), built-in `gx.*` capabilities, `gx.patch_file` marker edits, worked examples, and pitfalls — read `skills/gx-engineering/SKILL.md` (its `references/*.md` carry the depth).
- If the nested skill references files, resolve them relative to its own directory.
- Do not load every nested skill by default. Pick only the one matching the user's task.

## Workspace Assumptions

- `galaxy-flow` (gx) source: https://github.com/galaxio-labs/galaxy-flow
- Related collection: `gops` (galaxy-ops) skills live in https://github.com/galaxio-labs/gops-skills
- Prefer source and tests over stale docs when they disagree.

## Validation

- gx: `cargo build` / `cargo test` in `galaxy-flow`.
