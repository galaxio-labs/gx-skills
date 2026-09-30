---
name: gxl-authoring
description: "Use when writing or editing GXL (Galaxy Flow workflow language): mod/env/flow/fn/activity structures, flow heads and #[...] annotations, if/for control flow, variables and built-in constants, built-in gx.* capabilities, gx.patch_file marker edits, and GXL authoring pitfalls."
---

# GXL Authoring

How to write and edit GXL for `gx`. Deep references live in `references/`.
To run flows / manage projects from the CLI, see the separate **gx-cli** skill.

## Minimal template

```gxl
mod main {
  env default { ROOT = "./"; }
  flow conf { gx.echo(value: "hello galaxy flow"); }
}
```

## Structures

`mod` (with `mod name : base` inheritance), `env`, `flow`, `fn`, `activity`, plus `extern mod` for external modules.

```gxl
extern mod net { git = "https://example.com/repo.git", branch = "main"; }

mod sys {
  fn echo_tag(tag = "INFO", *msg) { gx.echo(value: "[${tag}] ${msg}"); }
  activity copy { src = ""; dst = ""; executer = "copy_act.sh"; }
}

mod main : sys {
  flow conf { sys.echo_tag(msg: "start"); sys.copy(src: "a.txt", dst: "b.txt"); }
}
```

## Flow heads

```gxl
flow test : pre1,pre2 : post1,post2 { ... }        # colon form (pre/post)
flow pre1 | pre2 | @test | post1 | post2 { ... }   # pipe form, `@` marks the main flow
flow test | post1 | post2 { ... }                  # shorthand pipe
```

## Annotations (`#[...]`, `name="value"`)

`#[usage(desp="...", color="...")]`, `#[auto_load(entry)]` / `#[auto_load(exit)]`, `#[task(name="...")]`; `#[fun]` (no args) is valid.

## Calls

`gx.<name>(key: "value", ...)`, `,`-separated; an anonymous first arg maps to `default`:

```gxl
gx.cmd(cmd: "echo hello");
gx.echo("hello");            # == gx.echo(value: "hello")
```

> Do **not** use the old block form `gx.xxx { ... }` — not supported.

## Control flow

```gxl
if defined(${DEPLOY}) && ${DEPLOY} == "true" {
  gx.echo(value: "deploy");
} else if ${STAGE} == "test" {
  gx.echo(value: "test");
} else {
  gx.echo(value: "skip");
}

for ${CUR} in ${DATA} { gx.echo(value: "item=${CUR}"); }
```

Operators: comparison `== != > >= < <= =*` (`=*` wildcard); logic `&& || !`; function `defined(${VAR})`.

## Variables & built-in constants

- Assign `NAME = value;` — string (`"t"` / `r#"raw"#`), bool, number, object `{ K: V }`, list `[V, ...]`; refs `${VAR}`, `${OBJ.KEY}`, `${ARR[0]}`. Keys are **case-insensitive**.
- Env-declared variables are surfaced with an `ENV_` prefix (docs: `${ENV_DATA_LIST}`).
- Auto-injected constants: `GXL_PRJ_ROOT`, `GXL_GIT_BRANCH`, `GXL_START_ROOT`, `GXL_CUR_DIR`, `GXL_CMD_ARG`, `GXL_CMD_DRYRUN`, `GXL_CMD_MODUP`, `GXL_OS_SYS`.

Full grammar (EBNF) and details: `references/gxl-syntax.md`.

## Built-in capabilities (`gx.*`)

`gx.assert`, `gx.cmd`, `gx.echo`, `gx.read_file` / `gx.read_cmd` / `gx.read_stdin`,
`gx.vars` (**env-block only**), `gx.tpl`, `gx.ver`, `gx.sn`, `gx.run`, `gx.shell`,
`gx.tar` / `gx.untar`, `gx.download` / `gx.upload`, `gx.patch_file`.

Expression functions (in `if`, not commands): `defined(${VAR})`, `gx.exists(flow: "...")`.
`gx.artifact` is documented but **not** wired as a block built-in; legacy `rg.*` aliases should be avoided.

Signatures, parameters and examples: `references/gxl-builtins.md`.

## Authoring `gx.shell` / `gx.cmd`

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
`@gxl:set(id)` / `@gxl:line(id)` / `@gxl:block(id)` + `@gxl:end(id)` (`#@gxl:*` / `@gxl:*` / `//@gxl:*` all match).

```gxl
gx.patch_file(file: "./Cargo.toml", action: "set", marker: "version", value: "0.12.3");
```

Details: `references/patch-file.md`.

## Pitfalls

- Use the call form `gx.<name>(...)`; the legacy block form `gx.xxx { ... }` is not supported.
- `${VAR}` interpolation only; bare shell `$var` / `$(...)` pass through to the shell — don't assume GXL expansion.

## References

- `references/gxl-syntax.md` — GXL structures, flow heads, annotations, control flow, variables, built-in constants (EBNF skeleton).
- `references/gxl-builtins.md` — every `gx.*` built-in + expression functions: purpose, parameters, examples, gotchas.
- `references/gxl-examples.md` — worked examples (assert, dryrun, fun, read, shell, template, transaction, vars).
- `references/patch-file.md` — `gx.patch_file`: marker model, parameters, MVC design, constraints.
