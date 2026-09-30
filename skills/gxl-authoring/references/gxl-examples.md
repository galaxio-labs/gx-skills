# GXL Examples Reference

Condensed reference derived from `galaxy-flow/docs/gxl/example/` (one `##` per
doc). Snippets are copied from the docs (fences normalized to `gxl`, trimmed).

## Index (`index.md`)

The example set: `assert`, `dryrun`, `fun`, `read`, `shell`, `template`, `transaction`, `vars`.

## Assert (`assert.md`)

`gx.assert` checks variable values, including object/array access. Companion
flows use `#[auto_load(entry|exit)]`; a `base.define` flow asserts module
variables like `${BASE_MOD_VAL}` and `${BASE}`. `base_env`/`envs` define shared
envs (`_common`, `cli`, `unit_test`, `default`, `empty`, `ut`).

```gxl
flow assert_main {
  sys_a = { mod1 : "A", mod2 : "B" };
  sys_b =  [ "C", "D" ];
  sys_d = ${SYS_A.MOD1} ;
  gx.assert ( value : "${MAIN_CONF}" , expect : "${ENV_ROOT}/conf" );
  gx.assert ( value : "${sys_a.mod1}" , expect : "A" );
  gx.assert ( value : "${sys_b[0]}" , expect : "C" );
  gx.assert ( value : "${sys_d}" , expect : "A" );
}
```

## Dryrun (`dryrun.md`)

`#[dryrun(...)]`: when a flow's assertion fails the annotated flow runs instead.

```gxl
#[dryrun(_step3)]
flow _step2 {
    gx.echo ("step2");
    gx.assert ( value : "true" , expect : "false" );
}

flow start | _step1 | _step2 ;
```

## Function (`fun.md`)

`fn` helpers defined in one module and called from a flow. `DATA` (list) and `OBJ`
(object) are env variables from `mod envs`.

```gxl
mod sys {
  fn echo(name) {
    gx.echo(value: "echo:${name}");
  }

  fn echo_obj(obj) {
    gx.echo(value: "echo_obj:${obj}");
  }

  fn echo_list(list) {
    gx.echo(value: "echo_list:${list}");
  }
}

mod main {
  flow conf {
    sys.echo(name: "test");
    sys.echo_obj(obj: "${OBJ}");
    sys.echo_list(list: "${DATA}");
  }
}
```

## Read (`read.md`)

Reads data with `gx.read_file` (ini/yaml/json) and `gx.read_cmd`; `stream: "true"`
forwards output live, but `gx.read_cmd` still writes only stdout into the
variable. It also iterates the read data.

```gxl
gx.read_cmd (
    cmd : "PYTHONUNBUFFERED=1 ansible-playbook site.yml -vv",
    name : "LAST_STDOUT",
    stream : "true" );

gx.read_file ( file : "./var2.ini" , name : "DATA");

for ${CUR} in ${DATA} {
    gx.echo ( value : "${CUR}" );
}
```

## Shell (`shell.md`)

Runs scripts/commands with `gx.shell`, capturing output via `out_var` and passing
an argument file via `arg_file`. `stream: "true"` suits long-running commands
like ansible; loops run shell over list and object data.

```gxl
gx.shell(
    arg_file: "./var.json",
    shell: "./demo.sh",
    out_var: "SYS_OUT");

gx.shell(
    shell: "PYTHONUNBUFFERED=1 ansible-playbook site.yml -vv",
    stream: "true");

gx.read_file(file: "./var_list.yml", name: "DATA");
for ${CUR} in ${DATA.DEV_LANG} {
    gx.shell(
        shell: "./demo_ex.sh ${CUR}",
        out_var: "SYS_OUT");
}
```

## Template (`template.md`)

Renders templates with `gx.tpl` (tpl dir + value file → dst dir), after preparing
the destination with `os.path`, plus a plain `gx.cmd` file copy.

```gxl
mod main {
  conf = "${ENV_ROOT}/conf";

  flow conf {
    os.path(dst: "${MAIN_CONF}/used", keep: "true");

    gx.tpl(
      tpl: "${MAIN_CONF}/tpls",
      dst: "${MAIN_CONF}/used",
      file: "${MAIN_CONF}/value.json"
    );

    gx.cmd(cmd: "cp ./conf/value.json ./conf/used/back.json");
  }
}
```

## Transaction (`transaction.md`)

Transactional flows: `#[transaction, undo(_undo_step1)]`, steps tagged
`#[undo(...)]`, and flow chains. `step3` fails its assertion, triggering undo.

```gxl
flow trans1 | step1 | step2 | base.base_step1 | step3;

#[transaction, undo(_undo_step1)]
flow step1 {
    gx.echo(" step1 ");
}
#[undo(_undo_step2)]
flow step2 {
    gx.echo(" step2 ");
}
#[undo(_undo_step3)]
flow step3 {
    gx.echo(" step3 ");
    gx.assert(value: "true", expect: "false");
}

flow _undo_step1 {
    gx.echo(" undo step1 ");
}
flow _undo_step2 {
    gx.echo(" undo step2 ");
}
flow _undo_step3 {
    gx.echo(" undo step3 ");
}
```

## Vars (`vars.md`)

Defines list and object env variables and loops over them, accessing object fields with `${CUR.NAME}`.

```gxl
mod envs {
  env default {
    DATA_LIST = ["JAVA", "RUST", "PYTHON"];
    DATA_OBJ = {
      JAVA: { NAME: "JAVA", SCORE: 80 },
      RUST: { NAME: "RUST", SCORE: 100 },
      PYTHON: { NAME: "PYTHON", SCORE: 200 }
    };
  }
}

mod main {
  flow array_do {
    for ${CUR} in ${ENV_DATA_LIST} {
      gx.echo(value: "CUR:${CUR}");
    }
  }

  flow obj_do {
    for ${CUR} in ${ENV_DATA_OBJ} {
      gx.echo(value: "CUR:${CUR.NAME}:${CUR.SCORE}");
    }
  }
}
```
