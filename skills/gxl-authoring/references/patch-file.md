# `gx.patch_file` Reference

`gx.patch_file` performs controlled, marker-based edits to a file from a GXL
workflow. Instead of blind text search-and-replace (unstable, hard to audit),
it locates change sites by **comment markers** and applies a fixed set of
actions: value substitution, line comment/uncomment, block comment/uncomment.

Sources: `galaxy-flow/docs/gxl/inner/patch_file.md` and
`galaxy-flow/docs/design/gx-patch-file-mvc.md`; implementation in
`src/ability/patch/{model,view,controller}.rs`. Status: implemented (MVP).

## Syntax

```gxl
gx.patch_file(
  file: "./Cargo.toml",
  action: "set",
  marker: "version",
  value: "0.12.3",
  strict: "true",
  dry_run: "false",
  backup: "false",
  comment_prefix: "#"
);
```

## Parameters

| Parameter | Required | Default | Notes |
| --- | --- | --- | --- |
| `file` | yes | — | Target file path. |
| `action` | yes | — | One of `set`, `comment_line`, `uncomment_line`, `comment_block`, `uncomment_block`. |
| `marker` (or `id`) | yes | — | Marker id. |
| `value` | only for `set` | — | New value written into the `set` target. |
| `strict` | no | `true` | Strict validation (see below). |
| `dry_run` | no | `false` | Compute changes without writing to disk. |
| `backup` | no | `false` | Back up to `<file>.bak` before writing. |
| `comment_prefix` (or `comment`) | no | `#` | Comment prefix inserted/removed by comment actions. |

Values are passed as strings; booleans use `"true"` / `"false"`.

## Marker model

- set target line: `@gxl:set(<id>)`
- line target line: `@gxl:line(<id>)`
- block start: `@gxl:block(<id>)`
- block end: `@gxl:end(<id>)`

The `<id>` is the `marker` parameter value. Markers are comments, and both `#`
and `//` prefixes are supported, including the tight (no-space) form:

```text
# @gxl:...
#@gxl:...
// @gxl:...
//@gxl:...
```

## Actions

- **`set`** — only lines containing `@gxl:set(id)`. Locates the assignment
  separator (`=` or `:`) on that line and replaces the value segment.
  Preserves the original indentation, the key and separator, and the trailing
  marker comment.
- **`comment_line` / `uncomment_line`** — only lines containing
  `@gxl:line(id)`. Comment inserts `comment_prefix` after the indentation;
  uncomment removes one `comment_prefix` (plus an optional following space).
- **`comment_block` / `uncomment_block`** — operates on the content lines
  between `@gxl:block(id)` and `@gxl:end(id)`. The marker lines themselves are
  left unchanged by default; the lines in between are commented/uncommented.

## Safety and constraints

- **`strict` (default `true`)** — any malformed marker structure fails the
  run. Examples:
  - marker hit count is not 1;
  - nested blocks;
  - an `end` with no matching `block`;
  - a missing `end`.
  A valid `block` must be exactly one group, with the start before the end.
  When strict validation fails, execution stops with an error.
- **`dry_run`** — computes and reports changes but does not write the file.
- **`backup`** — takes effect only when `dry_run = false` and there are actual
  changes; writes `<target>.bak` first.
- **`comment_prefix`** — must not be an empty string. This is validated both at
  parse time and at runtime to avoid accidental writes.
- `set` without `value` fails at runtime; empty `comment_prefix` fails at parse
  time.

## Examples

### `set`

```toml
version = "0.12.2"   # @gxl:set(version)
```

```gxl
gx.patch_file(
  file: "./Cargo.toml",
  action: "set",
  marker: "version",
  value: "\"0.12.3\""
);
```

`set` also accepts `:` as the separator: given
`image: app:v1   # @gxl:set(image)` with `value: "app:v2"`, the line becomes
`image: app:v2   # @gxl:set(image)`.

### Block comment / uncomment

```toml
# @gxl:block(res_depend_test)
[features]
res_depend_test = []
# @gxl:end(res_depend_test)
```

```gxl
gx.patch_file(file: "./Cargo.toml", action: "comment_block", marker: "res_depend_test");
gx.patch_file(file: "./Cargo.toml", action: "uncomment_block", marker: "res_depend_test");
```

`comment_block` result:

```toml
# @gxl:block(res_depend_test)
#[features]
#res_depend_test = []
# @gxl:end(res_depend_test)
```

### Line toggle

```toml
res_depend_test = []   # @gxl:line(res_depend_test)
```

```gxl
gx.patch_file(file: "./Cargo.toml", action: "comment_line", marker: "res_depend_test");
gx.patch_file(file: "./Cargo.toml", action: "uncomment_line", marker: "res_depend_test");
```

### Custom `comment_prefix`

```gxl
gx.patch_file(
  file          : "./src/main.rs",
  action        : "comment_line",
  marker        : "demo_marker",
  comment_prefix: "//"
);
```

### `dry_run` + `backup` + `strict`

```gxl
gx.patch_file(
  file   : "./Cargo.toml",
  action : "comment_block",
  marker : "res_depend_test",
  strict : "true",
  dry_run: "true",
  backup : "true"
);
```

## MVC design summary

- **Model** (`src/ability/patch/model.rs`) — `PatchAction` enum
  (`Set`, `CommentLine`, `UncommentLine`, `CommentBlock`, `UncommentBlock`)
  and the `GxPatchFile` request model (`file`, `marker`, `action`, `value`,
  `strict`, `dry_run`, `backup`, `comment_prefix`).
- **Controller** — split in two: a DSL controller
  (`src/parser/inner/patch.rs`) parses `gx.patch_file(...)` into `GxPatchFile`;
  the runtime controller (`src/ability/patch/controller.rs`) reads the file,
  applies the patch per `action + marker`, runs strict validation, writes the
  file according to `dry_run`/`backup`, and builds the result.
- **View** (`src/ability/patch/view.rs`) — `PatchView` emits a uniform summary
  (`action`, `file`, `marker`, `marker_hits`, `changed_lines`, `dry_run`) into
  `Action.stdout` for task records and observability.

Non-goals: no AST-level rewriting (line-text patching); no automatic conflict merging.

## Gotchas

- Patching is line/text level, not AST level — place markers deliberately.
- Under `strict`, every `set`/`line` marker must match exactly once, and block
  markers must pair uniquely and never nest.
- `dry_run` never touches disk; `backup` only applies to a real write with
  actual changes.
- An empty `comment_prefix` is rejected (parse-time and runtime).
- `action` and boolean parameters are parsed as strings.
