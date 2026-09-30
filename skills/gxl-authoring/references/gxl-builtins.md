# GXL Built-in Blocks (`gx.*`) and Expression Functions

Condensed reference for the capabilities the current GXL block parser supports
directly, plus the two expression functions. Built-ins are called as
`gx.<name>(...)` with `key: "value"` string parameters (booleans use
`"true"`/`"false"`); most also accept an anonymous first argument (`gx.echo("hi")`).

Source: `galaxy-flow/docs/gxl/inner/*.md`. **Always use `gx.*`.** Legacy `rg.*`
parse-compatibility entry points exist, but they are not uniformly opened at the
block/env statement-dispatch layer, so `rg.*` is unreliable and should be avoided.

## At a glance

| Capability | Kind | Purpose |
| --- | --- | --- |
| `gx.assert` | block | Assert a comparison (equal by default). |
| `gx.cmd` | block | Run one command string. |
| `gx.echo` | block | Print text to stdout. |
| `gx.read_file` | block | Read `ini`/`json`/`yml` config into variables. |
| `gx.read_cmd` | block | Run a command and store its stdout in a variable. |
| `gx.read_stdin` | block | Read a value from stdin. |
| `gx.vars` | block (env only) | Batch-define variables inside an `env`. |
| `gx.tpl` | block | Render a template (handlebars). |
| `gx.ver` | block | Read/increment a version number and export it. |
| `gx.sn` | block | Read/update a numeric serial-number file and export it. |
| `gx.run` | block | Run another GXL config in a subdirectory. |
| `gx.shell` | block | Run a shell command/script; load args; capture output. |
| `gx.tar` / `gx.untar` | block | Create / extract a tar archive. |
| `gx.download` / `gx.upload` | block | HTTP(S) file transfer. |
| `gx.patch_file` | block | Controlled marker-based file edits. |
| `defined(${VAR})` | expression | True if the variable is defined. |
| `gx.exists(flow: "...")` | expression | True if the named flow exists. |

Not wired as a block built-in: `gx.artifact` (see below).

## gx.assert
Assert a comparison; by default it asserts "equal". Params: `value` (actual),
`expect` (expected), `err` (failure message, optional), `result` (`"true"` =
expect equal [default]; `"false"` = expect not equal).

```gxl
gx.assert(value: "${ENV}", expect: "prod");
gx.assert(value: "${ENV}", expect: "prod", result: "false");
```

## gx.cmd
Run one command string. By default the result is printed after the command
finishes; use `stream: "true"` to watch long-running commands live. Params:
`cmd` (command; also anonymous first arg), `err` (error var), `suc` (success
message), `ok_codes` (accepted exit codes, e.g. `"0,2"`), `sudo`, `log`
(`1`/`2`/`3`), `silence`, `quiet`, `stream` (default `"false"`).

```gxl
gx.cmd(cmd: "git branch --show-current");
gx.cmd(cmd: "grep foo missing.txt", ok_codes: "0,1");
gx.cmd(cmd: "ansible-playbook site.yml -vv", stream: "true");
```

A code-block form is also supported (` ```cmd ` with one command per line).

Gotchas (`stream`):

- `"false"` (default) prints/returns stdout/stderr only after the command
  finishes; `"true"` forwards them live while still retaining the full output for
  the `Action` record. `stream` does not change exit-code judging (still `ok_codes`).
- With `quiet: "true"`, output is not printed live to the terminal, but the
  command still reads in streaming mode and output is retained.
- Terminal interleaving order of stdout/stderr is not guaranteed; merge the
  streams in the shell yourself if a fixed order is required. Intended for long
  commands (`ansible-playbook`, `terraform apply`, `kubectl rollout status`).

## gx.echo
Print text to stdout. Params: `value` (text; also anonymous first arg).

```gxl
gx.echo(value: "hello");   # gx.echo("hello");
```

The current implementation prints text only; the old `file`/`export`/`inc`
parameters are **not** supported.

## gx.read_file / gx.read_cmd / gx.read_stdin
- **`gx.read_file`** — read a config file (`ini`, `json`, `yml`) into the variable
  space. Params: `file` (path; also anonymous first arg), `name` (optional; when
  omitted, object fields merge into global variables), `entity` (accepted by the
  parser but **not used** by the execution layer).
- **`gx.read_cmd`** — run a command and write its `stdout` into a variable. Params:
  `name` (variable), `cmd`, `err`, `ok_codes` (e.g. `"0,1"`), `log`, `stream`.
  Only `stdout` is written; with `stream: "true"` stdout/stderr stream live during
  execution but the variable still receives only stdout.
- **`gx.read_stdin`** — read a value from stdin. Params: `prompt`, `name`.

```gxl
gx.read_file(file: "./var.yml", name: "DATA");
gx.read_cmd(name: "BRANCH", cmd: "git branch --show-current", ok_codes: "0,1");
gx.read_stdin(prompt: "input your name:", name: "USER_NAME");
```

## gx.vars
Batch-define variables. **`gx.vars` is env-block only** — used only as an env
item inside `env`. The current syntax is `=` assignment with `;`/`,` separators.

```gxl
env default {
  gx.vars {
    APP = "galaxy";
    STAGE = "dev";
  };
}
```

## gx.tpl
Template rendering (currently only the `handlebars` engine). Params: `tpl`
(template file or directory), `dst` (target file or directory), `data` (inline
JSON string, optional), `file` (JSON data file, optional), `engine`
(`handlebars` | `helm`; the execution layer only supports `handlebars`).

```gxl
gx.tpl(tpl: "./conf/tpls", dst: "./conf/used", file: "./conf/value.json");
```

## gx.ver
Read and increment a version number (writes it back to the file and exports it).
Params: `file`, `inc` (`build` | `bugfix` | `feature` | `main` | `null`).
Supported formats: `major.minor.patch` and `major.minor.patch.build`.

```gxl
gx.ver(file: "./version.txt", inc: "build");
```

Default exported variable is `VERSION`. The `export` parameter has a parser-layer
implementation deviation and should not be used.

## gx.sn
Read or update a numeric serial-number file and export the number. Params:
`file`, `action` (`read` [default] | `add` | `reset`), `export` (exported
variable name; default `SN`).

```gxl
gx.sn(file: "./sn.txt");
gx.sn(file: "./sn.txt", action: "add");
gx.sn(file: "./sn.txt", export: "BUILD_SN", action: "add");
```

- `read` (default) reads the number and exports it, without writing back.
- `add` reads the current number, writes back and exports `current + 1`.
- `reset` writes back and exports `1`; it can create the file if missing.
- The file content must be an integer `>= 1`.

## gx.run
Run another GXL config in a subdirectory. Params: `local` (run dir), `conf`
(target config; default `./_gal/work.gxl`), `env` (target environment), `flow`
(target flow(s); comma-separated to forward several, one by one), `isolate`
(`"true"`/`"false"` — isolate the variable space).

```gxl
gx.run(local: ".", env: "default", flow: "localize");
```

**`flow` is required**, and `local`/`env` must be supplied together with it.

## gx.shell
Run a shell command/script; can load an argument file and write back an output
variable. Params: `shell` (command or script; also anonymous first arg),
`arg_file` (`json` | `yml` | `yaml` | `toml` | `ini`), `out_var`, `err`,
`ok_codes` (e.g. `"0,2"`), `log`, `sudo`, `silence`, `stream`.

```gxl
gx.shell(shell: "PYTHONUNBUFFERED=1 ./deploy.sh", out_var: "DEPLOY_OUT");
gx.shell(shell: "PYTHONUNBUFFERED=1 ansible-playbook site.yml -vv", stream: "true");
```

`stream: "true"` streams stdout/stderr live without affecting `out_var`, the exit
code, or the internal result record. If the script buffers its own output, set
the corresponding unbuffered flag (e.g. `PYTHONUNBUFFERED=1`).

## gx.tar / gx.untar
Params: `gx.tar` — `src`, `file`; `gx.untar` — `file`, `dst`. `gx.untar` cleans
the target path (if it exists) before extracting.

```gxl
gx.tar(src: "./src", file: "./dist/src.tar.gz");
gx.untar(file: "./dist/src.tar.gz", dst: "./dist/unpack");
```

## gx.download / gx.upload
Params: `gx.download` — `url`, `local_file`, `username`, `password`, `force`;
`gx.upload` — `url`, `local_file`, `method`, `username`, `password`.

```gxl
gx.download(url: "https://example.com/a.txt", local_file: "./temp/a.txt", force: "true");
gx.upload(url: "https://example.com/upload", local_file: "./temp/a.txt", method: "put");
```

- The parent directory of `local_file` must already exist.
- `gx.download`: if `local_file` is a directory, the file uses the URL filename.
- `force` (optional, default `"false"`): when the local file exists it is skipped
  by default (`reuse_cache`); `force: "true"` ignores it and re-downloads (via
  `UpdateScope::RemoteCache`, which clears the cache for that address).
- An interrupted download leaves no partial file: it writes `<local_file>.part`
  first and atomically replaces on success (temp file removed on failure).
- On transfer failures (network jitter / 5xx / truncation) up to 3 internal
  retries; 4xx and local IO errors are not retried.

## gx.patch_file
Controlled marker-based file edits supporting `set`, `comment_line`,
`uncomment_line`, `comment_block`, `uncomment_block`. See the dedicated
`patch-file.md` reference for the full parameter table, marker model, and
`strict`/`dry_run`/`backup` semantics.

```gxl
gx.patch_file(file: "./Cargo.toml", action: "set", marker: "version", value: "0.12.3");
```

## `defined(${VAR})` (expression function)
True if the variable is defined. It is an **expression function, not a
`gx.defined` command** — write it directly in an `if` condition.

```gxl
if defined(${HOME}) { gx.echo(value: "has home"); }
if !defined(${NO_SUCH_VAR}) { gx.echo(value: "missing"); }
```

## `gx.exists(flow: "...")` (expression function)
True if the **flow** exists. It sits at the same level as `defined(${VAR})` —
written directly in `if` — for "call if present, skip if absent".

```gxl
if gx.exists(flow: "localize") {
    gx.run(local: ".", env: "default", flow: "localize");
}
```

- `flow`: the flow name; also accepts a bare value `gx.exists("localize")`, a
  variable `gx.exists(${P})`, or negation `!gx.exists(...)`. An unqualified name
  defaults to the `main` module; a `mod.flow` qualified name is also accepted
  (e.g. `gx.exists("ops-mod.install")`).
- Judgment is based on the **flow-name set of the currently assembled space**, so
  it does **not** resolve extern, does **not** go online, and has **no side
  effects**. A missing flow returns `false` (not an error), independent of any
  exit code.
- CLI equivalence: `gx run <flow> --exists` (exit `0` if the flow exists, `1` if
  not), meant for use **outside** gx (scripts or higher-level tools such as
  `galaxy-ops`). It only considers flows declared in the conf, is **independent of
  `-e/--env`**, loads the conf without running any flow, and treats a conf that
  fails to load (including an uncached extern module) as "does not exist", exiting
  `1` with the reason on stderr.

## `gx.artifact` (documented, not wired)
`gx.artifact` is documented but is **not** recognized as a built-in block by the
current `BlockAction`/`stc_blk`. Implement artifact flows via an external
module/activity call, e.g. `os.artifact(file: "./target/a", dst: "./artifacts")`;
the docs recommend keeping this in `_gal/mods` as a module call.
