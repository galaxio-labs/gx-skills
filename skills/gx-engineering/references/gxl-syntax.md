# GXL Syntax Reference

Condensed from `galaxy-flow/docs/gxl/{syntax,var_def,const,help}.md`.

## File structure

A GXL file is a sequence of `mod` blocks, optionally preceded by `extern mod`:

```gxl
extern mod os,ssh { path = "./_gal/mod"; }
extern mod net { git = "https://example.com/repo.git", branch = "main"; }

mod main {
  env default {
    ROOT = "./";
  }

  flow conf {
    gx.echo(value: "hello");
  }
}
```

Grammar skeleton (from `docs/gxl/syntax.md`):

```ebnf
GxlFile      = { ExternMod | Module } ;
ExternMod    = "extern" "mod" ModNameList ModAddr ;
ModAddr      = "{" ( "path" "=" String
                 | "git" "=" String [ "," ("branch"|"channel") "=" String ] [ "," "tag" "=" String ] ) "}" ;
Module       = [Annotation] "mod" Name [":" MixList] "{" { Prop | Env | Flow | Fun | Activity } "}" [";"] ;
Env          = [Annotation] "env" Name [":" MixList] (";" | "{" { Prop | EnvStmt } "}" [";"]) ;
Flow         = [Annotation] "flow" FlowHead (";" | Block [";"]) ;
Fun          = "fn" Name "(" [FunParams] ")" (";" | Block [";"]) ;
Activity     = "activity" Name "{" { FormalParam [","|";" ] } "}" ;
Block        = "{" { Prop | IfStmt | ForStmt | Builtin | Call | CmdBlock } "}" ;
IfStmt       = "if" Expr Block { "else" "if" Expr Block } [ "else" Block ] ;
ForStmt      = "for" VarRef "in" VarRef Block ;
CmdBlock     = "```cmd" <raw text> "```" ;
Annotation   = "#[" AnnFun {"," AnnFun} "]" ;
```

## Blocks: `mod` / `env` / `flow` / `fn` / `activity`

```gxl
mod sys {
  fn echo_tag(tag = "INFO", *msg) {
    gx.echo(value: "[${tag}] ${msg}");
  }

  activity copy {
    src = "";
    dst = "";
    executer = "copy_act.sh";
  }
}

mod main : sys {
  flow conf {
    sys.echo_tag(msg: "start");
    sys.copy(src: "a.txt", dst: "b.txt");
  }
}
```

## Flow head forms

```gxl
flow test : pre1,pre2 : post1,post2 { ... }   # colon form (pre/post)
flow pre1 | pre2 | @test | post1 | post2 { ... }  # pipe form, `@` marks the main flow
flow test | post1 | post2 { ... }             # shorthand pipe
```

## Annotations (`#[...]`)

```gxl
#[usage(desp="developer local env", color="red"), auto_load(entry)]
flow __into { ... }
```

- `#[usage(desp="...", color="...")]` — menu/description metadata.
- `#[auto_load(entry)]` / `#[auto_load(exit)]` — auto-run entry/exit flow.
- `#[task(name="...")]` — task name for a flow.
- Argument form is `name="value"`; `#[fun]` (no args) is also valid.

## Call syntax

All built-ins are function calls — `gx.<name>(key: "value", ...)`, `,`-separated.
Most accept an anonymous first argument mapped to `default`:

```gxl
gx.cmd(cmd: "echo hello");
gx.echo("hello");            # equivalent to gx.echo(value: "hello")
```

`gx.cmd` / `gx.shell` / `gx.read_cmd` accept `stream: "true"` for live output of long commands.

> Do **not** use the old block form `gx.xxx { ... }` — it is not supported.

## Control flow

```gxl
if defined(${DEPLOY}) && ${DEPLOY} == "true" {
  gx.echo(value: "deploy");
} else if ${STAGE} == "test" {
  gx.echo(value: "test");
} else {
  gx.echo(value: "skip");
}

for ${CUR} in ${DATA} {
  gx.echo(value: "item=${CUR}");
}
```

Operators: comparison `== != > >= < <= =*` (`=*` wildcard); logic `&& || !`; function `defined(${VAR})`.

## Variables

```gxl
flow conf {
  ONE   = "one";
  SYS_A = { MOD1: "A", MOD2: "B" };
  SYS_B = ["C", "D"];
  SYS_C = ${SYS_B[1]};
  SYS_D = ${SYS_A.MOD1};
}
```

- Types: string (`"text"` / `r#"raw"#`), bool, number, object `{ K: V }`, list `[V, ...]`, refs `${VAR}` / `${OBJ.KEY}` / `${ARR[0]}`.
- **Keys are case-insensitive** at runtime (normalized to upper case): `${SYS_A.MOD1}` == `${sys_a.mod1}`.
- Variables declared in an `env` are surfaced to flows with the `ENV_` prefix (docs show `${ENV_DATA_LIST}`):

```gxl
mod envs {
  env default {
    DATA_LIST = ["JAVA", "RUST", "PYTHON"];
  }
}
mod main {
  flow list_do {
    for ${CUR} in ${ENV_DATA_LIST} { gx.echo(value: "CUR=${CUR}"); }
  }
}
```

## Built-in constants (auto-injected)

| Constant | Meaning |
| --- | --- |
| `GXL_PRJ_ROOT` | Dir with `_gal/project.toml` (search upward); `UNDEFIN` if absent |
| `GXL_GIT_BRANCH` | Git branch from `GXL_PRJ_ROOT` / start dir; `UNDEFIN` if detached/non-git |
| `GXL_START_ROOT` | Working dir where `gx` was started |
| `GXL_CUR_DIR` | Current execution dir (differs from `GXL_START_ROOT` after `gx.run` changes dir) |
| `GXL_CMD_ARG` | CLI pass-through arg (`--cmd-arg`) |
| `GXL_CMD_DRYRUN` | CLI dry-run flag |
| `GXL_CMD_MODUP` | CLI module-update flag |
| `GXL_OS_SYS` | System id string (arch / os / major) |
