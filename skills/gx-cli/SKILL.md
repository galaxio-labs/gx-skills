---
name: gx-cli
description: "Use when running or managing gx (galaxy-flow) from the CLI: gx run / gx adm workflows, project init (gx init), module update (gx mod), built-in docs (gx doc), environment check, gx self self-update, gx self skill, the _gal/ layout, CLI flags, and how gx integrates with gops."
---

# Gx CLI

How to run and manage `gx` (the `galaxy-flow` CLI): workflows, project init, modules, self-update and skills.
To *author* the GXL that `gx` executes, see the separate **gxl-authoring** skill.

## Commands

- `gx run [flow]` — run a workflow (default `./_gal/work.gxl`).
- `gx adm [flow]` — run an admin flow (default `./_gal/adm.gxl`).
- `gx run <flow> --exists` — probe flow existence without running it (exit `0`/`1`).
- `gx init env` — initialize the runtime environment (`~/.galaxy/`, `conf.toml`, net access control).
- `gx init project [--repo URL] [--path subdir] [--branch B] [--tag T]` — init a project.
- `gx mod update` — update project modules declared by local `_gal` configs.
- `gx doc [topic] [--markdown]` — built-in docs, e.g. `gx doc gx.cmd`, `gx doc gx.patch_file`.
- `gx check` — print current runtime environment info.
- `gx self status|check|update|rollback` — self-update.
- `gx self skill install|list` — install / list agent skills (default source `galaxio-labs/gx-skills`).

Compatibility aliases (when symlinked): `grun …` ⇒ `gx run …`, `gadm …` ⇒ `gx adm …`.

## Flags

- `-e/--env <name>` — environment (default `default`)
- `-d/--debug <level>` — debug verbosity
- `-c/--conf <file>` — GXL config file
- `--log <module=level,...>` — log config
- `-q/--quiet`
- `--dryrun`
- `--ai` (currently degraded — see Pitfalls)

## `gx init project`

- No args → local init (offline): creates a basic `./_gal/work.gxl` and `./_gal/adm.gxl`.
- `--path <subdir>` (no `--repo`) → fetch that subdir from the default template repo `https://github.com/galaxio-labs/prj-tpl.git`.
- `--repo <git_url>` → fetch from that repo (optionally `--path <subdir>`).
- `--branch` and `--tag` are mutually exclusive; both require `--repo` or `--path`.

## Directory conventions

- `./_gal/work.gxl` — workflow entry (`gx run`)
- `./_gal/adm.gxl` — admin flow entry (`gx adm`)
- `./_gal/mods/` — local project modules
- `gx run` / `gx adm` read `_gal/*.gxl` relative to the **current** directory — run from the project root.

## Self-update (`gx self`)

```bash
gx self status
gx self check --channel <stable|alpha|beta>
gx self update --channel <stable|alpha|beta> [--to <version>] [--dry-run] [--force] --yes
gx self rollback [--id <backup_id>]
```

- `rollback` takes `--id` (not `--backup-id`).
- The download goes to a temp dir and is cleaned up; a successful update replaces the `gx` in the install dir.
- Backups/state: `~/.galaxy/self_update/state.json`, `~/.galaxy/self_update/backups/<backup_id>/`.
- Manifest: `https://raw.githubusercontent.com/galaxio-labs/get/main/updates/gx/{channel}/manifest.json` (same source as `inst-x.sh gx <channel>`).

## Agent skills (`gx self skill`)

```bash
gx self skill install           # whole collection; auto-detects installed platforms
gx self skill install <skill>   # one skill under skills/
gx self skill list              # list installable skills
```

- Default source `galaxio-labs/gx-skills@main`; `--source` accepts `owner/repo`, a git URL, or a local checkout; `--ref` picks a branch/tag.
- `--target codex|claude|zed|all` (repeatable) and `--dir <path>` (repeatable) choose destinations; with neither, it auto-detects `~/.codex/skills`, `~/.claude/skills`, `~/.agents/skills`, falling back to all three.
- Every `SKILL.md` frontmatter is validated before installing; `--symlink` works for local sources only.

## Relationship to gops

- `galaxy-flow` (`gx`) defines and executes workflows; `galaxy-ops` (`gops`) organizes and delivers modules, systems and projects.
- A `gxl`-type system in gops dispatches `run start/stop/...` to `gx` (specifically `gx run -e <env> -d <debug> [--cmd-arg <mod>] <cmd>`), and `gops` requires `gx >= 0.13.0` at `$HOME/bin/gx`.
- gops-side usage (modules, systems, ops projects) is covered by the separate **gops-skills** collection: https://github.com/galaxio-labs/gops-skills

## Pitfalls

- `gx run`/`gx adm` read `./_gal/*.gxl` relative to the current directory; run from the project root.
- `gxl` dispatch shells out to `$HOME/bin/gx` — a stale `gx` on PATH silently keeps old behaviour.
- AI ability is currently degraded (`ai_diagnose` is a no-op; `gx.ai_chat` is not a built-in block ability).
