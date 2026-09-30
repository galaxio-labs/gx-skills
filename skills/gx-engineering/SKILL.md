---
name: gx-engineering
description: "Use when working with gx (galaxy-flow): running GXL workflows (gx run / gx adm), initializing projects, authoring GXL (modules, envs, flows, fn/activity, built-in gx.* capabilities, gx.patch_file marker edits), and troubleshooting gx execution."
---

# Gx Engineering

Use this skill for source-accurate work with `gx` (the `galaxy-flow` CLI). Deep
references live in `references/` next to this file (see [References](#references)).

## Commands

- `gx run [flow]` — run a workflow (default `./_gal/work.gxl`).
- `gx adm [flow]` — run an admin flow (default `./_gal/adm.gxl`).
- `gx run <flow> --exists` — probe flow existence without running it (exit `0`/`1`).
- `gx init env` — initialize the runtime environment (`~/.galaxy/`, `conf.toml`).
- `gx init project [--repo URL] [--path subdir] [--branch B] [--tag T]` — init a project.
- `gx mod update` — update project modules (`_gal`).
- `gx doc [topic] [--markdown]` — built-in docs (`gx doc gx.cmd`).
- `gx check` — print runtime environment info.
- `gx self status|check|update|rollback` — self-update.
- `gx self skill install|list` — install / list agent skills (default source `galaxio-labs/gx-skills`).

## Flags

- `-e/--env <name>` — environment (default `default`)
- `-d/--debug <level>` — debug verbosity
- `-c/--conf <file>` — GXL config file
- `--log <module=level,...>` — log config
- `-q/--quiet`
- `--dryrun`
- `--ai` (currently degraded — see Pitfalls)

## Directory conventions

- `./_gal/work.gxl` — workflow entry (`gx run`)
- `./_gal/adm.gxl` — admin flow entry (`gx adm`)
- `./_gal/mods/` — local project modules
- `gx run` / `gx adm` read `_gal/*.gxl` relative to the **current** directory — run from the project root.

## GXL essentials

Structures: `mod`, `env`, `flow`, `fn`, `activity` (plus `extern mod` for external modules).

```gxl
mod main {
  env default { ROOT = "./"; }
  flow conf { gx.echo(value: "hello"); }
}
```

- **Flow heads**: `flow test : pre : post` (colon) or `flow pre | @test | post` (pipe, `@` marks the main flow).
- **Annotations** (`#[...]`): `#[usage(desp=,color=)]`, `#[auto_load(entry|exit)]`, `#[task(name=)]`.
- **Calls**: `gx.<name>(key: "value", ...)`; an anonymous first arg maps to `default` (`gx.echo("hi")`). Do **not** use the old `gx.xxx { ... }` block form.
- **Control flow**: `if / else if / else`, `for ${CUR} in ${DATA}`; operators `== != > >= < <= =*` and `&& || !`, plus `defined(${VAR})`.
- **Variables**: `NAME = value;` with string/bool/number/object/list; refs `${VAR}`, `${OBJ.KEY}`, `${ARR[0]}`; keys are **case-insensitive**.
- **Built-in constants**: `GXL_PRJ_ROOT`, `GXL_GIT_BRANCH`, `GXL_START_ROOT`, `GXL_CUR_DIR`, `GXL_CMD_ARG`, `GXL_CMD_DRYRUN`, `GXL_CMD_MODUP`, `GXL_OS_SYS`.

Full details + the EBNF skeleton: `references/gxl-syntax.md`.

## Built-in capabilities (`gx.*`)

`gx.assert`, `gx.cmd`, `gx.echo`, `gx.read_file` / `gx.read_cmd` / `gx.read_stdin`,
`gx.vars` (**env-block only**), `gx.tpl`, `gx.ver`, `gx.sn`, `gx.run`, `gx.shell`,
`gx.tar` / `gx.untar`, `gx.download` / `gx.upload`, `gx.patch_file`.

Expression functions (used in `if`, not commands): `defined(${VAR})`, `gx.exists(flow: "...")`.
`gx.artifact` is documented but **not** wired as a block built-in; legacy `rg.*` aliases should be avoided.

Signatures, parameters and examples: `references/gxl-builtins.md`.

## Authoring GXL (`gx.shell` / `gx.cmd`)

- Both execute the string via `/bin/sh`, and **echo the whole command** before running it. Hide the echo with
  `silence: "true"` (keeps stdout); `quiet: "true"` does **not** hide it. `stream: "true"` forwards
  stdout/stderr live (use it for long-running commands).
- **No `+` string concatenation** — `"a" + "b"` (even split across lines) is a parse error (`need '}'`). Use one literal.
- **Interpolation:** only `${NAME}` / `${NAME:default}` are GXL-substituted; bare shell constructs
  (`$!`, `$$`, `$(...)`, `$((...))`, `$var`) pass through untouched, so both can mix in one string:
  `gx.shell ( "pid=$(cat ${pid_file}); kill $pid", silence: "true" );`.
- **Backgrounding a daemon:** `nohup <cmd> > <log> 2>&1 < /dev/null & echo $! > <pid>`. Beware `a && b &`:
  `&` binds the whole list, so `mkdir x && cmd &` backgrounds `mkdir` too — split with `mkdir x; …` first.
- `gx.read_file(file:, name:)` loads yaml/json/ini into a variable; `gx.download(url:, local_file:)` fetches a file.
- Envs: `mod envs { env <name> { VAR = "..."; } }`, selected with `gx run -e <name>`. From a **module root**
  the names are those in the root `_gal/work.gxl` (e.g. `arm_mac`, `x86_ubt`, `x86_ubt_k8s`); from a **model dir**
  the model's own `_gal/work.gxl` defines only `local`/`spec`/`default`, so a module-root env name there fails
  with `Environment '<name>' not found`.
- `mod_ops` dispatches an op with `gx.run( local: "./mod/${ENV_MODEL}", env: "${ENV_MODULE_ENV}", flow: "<op>" )`;
  module flows subclass `empty_operators` and tag overrides with `#[task(name="gops@<op>")]`.

## `gx.patch_file`

Marker-based, reviewable edits to a file (preferred over blind search-and-replace):
`gx.patch_file(file:, action:, marker:, value:, strict:, comment_prefix:)` with markers
`@gxl:set(id)` / `@gxl:line(id)` / `@gxl:block(id)` + `@gxl:end(id)` (`#@gxl:*` / `//@gxl:*` also match).

```gxl
gx.patch_file(file: "./Cargo.toml", action: "set", marker: "version", value: "0.12.3");
```

Details: `references/patch-file.md`.

## Relationship to gops

- `galaxy-flow` (`gx`) defines and executes workflows; `galaxy-ops` (`gops`) organizes and delivers modules, systems and projects.
- A `gxl`-type system in gops dispatches `run start/stop/...` to `gx` (specifically `gx run -e <env> -d <debug> [--cmd-arg <mod>] <cmd>`), and `gops` requires `gx >= 0.13.0` at `$HOME/bin/gx`.
- gops-side usage (modules, systems, ops projects) is covered by the separate **gops-skills** collection: https://github.com/galaxio-labs/gops-skills

## Pitfalls

- `gx run`/`gx adm` read `./_gal/*.gxl` relative to the current directory; run from the project root.
- Use the call form `gx.<name>(...)`; the legacy block form `gx.xxx { ... }` is not supported.
- AI ability is currently degraded (`ai_diagnose` is a no-op; `gx.ai_chat` is not a built-in block ability).

## References

- `references/gxl-syntax.md` — GXL structures, flow heads, annotations, control flow, variables, built-in constants (EBNF skeleton).
- `references/gxl-builtins.md` — every `gx.*` built-in + expression functions: purpose, parameters, examples, gotchas.
- `references/gxl-examples.md` — worked examples (assert, dryrun, fun, read, shell, template, transaction, vars).
- `references/patch-file.md` — `gx.patch_file`: marker model, parameters, MVC design, constraints.
