# Gx Skills

Focused skills for `gx` (galaxy-flow) — the GXL workflow engine and CLI.

The repo is organized as a top-level router skill plus nested tool-specific skills under `skills/`. gops (galaxy-ops) skills live separately in [gops-skills](https://github.com/galaxio-labs/gops-skills).

## Available Skills

- `gx-skills` — top-level router for gx tasks.
- `gx-cli` — running / managing `gx`: `run`/`adm`/`init`/`mod`/`doc`/`check`/`self`/`self skill`, `_gal/` layout, CLI flags, project init, self-update, and gops integration.
- `gxl-authoring` — writing GXL: `mod`/`env`/`flow`/`fn`/`activity`, flow heads, `#[...]` annotations, control flow, variables & built-in constants, built-in `gx.*` capabilities, `gx.patch_file`, worked examples, and authoring pitfalls. Deep material lives in `skills/gxl-authoring/references/`.

## Installation

### With `gx` (recommended)

If you have `gx` (>= 0.16.0) installed, use its native installer — no shell script, no `python3` / `ruby`:

```bash
gx self skill install           # whole collection; auto-detects installed platforms
gx self skill list              # list installable skills
```

`--source` accepts `owner/repo`, a git URL, or a local checkout; `--ref` selects a branch / tag; `--target codex|claude|zed|all` and `--dir <path>` choose destinations (both repeatable). Every `SKILL.md` frontmatter is validated before installing (built-in YAML parser, no external interpreter).

### With `gops`

If you have `gops` (>= 2.0.13) but not `gx` >= 0.16.0, install through it (the source must be given explicitly):

```bash
gops self skill install --source galaxio-labs/gx-skills
gops self skill list --source galaxio-labs/gx-skills
```

### With `install.sh`

Use this when you have neither `gx` >= 0.16.0 nor `gops` >= 2.0.13 (bootstrapping). Install the whole collection (router + nested skills):

```bash
./install.sh                          # install all available platforms
./install.sh --zed                    # only Zed (~/.agents/skills/)
./install.sh --dir ~/my/skills        # custom directory
```

Install a single skill by name (local checkout first, remote clone fallback):

```bash
./install.sh gxl-authoring --codex
./install.sh gx-cli --claude
```

Remote install (no local checkout):

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/galaxio-labs/gx-skills/main/install.sh) gxl-authoring
```

See `install.sh --help` for all options.

Before installing, `install.sh` validates the YAML frontmatter of every `SKILL.md` (prefers `python3` + PyYAML, falls back to `ruby` + psych). Invalid frontmatter aborts the install; if neither parser is available it warns and continues.

Environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `GX_SKILLS_REF` | Branch or tag to install | `main` |
| `GX_SKILLS_SOURCE` | Source GitHub repo | `galaxio-labs/gx-skills` |

## Layout

```text
gx-skills/
├── SKILL.md
├── README.md
├── CHANGELOG.md
├── install.sh
├── version.txt
├── agents/
│   └── openai.yaml
└── skills/
    ├── gx-cli/
    │   └── SKILL.md
    └── gxl-authoring/
        ├── SKILL.md
        └── references/
            ├── gxl-syntax.md
            ├── gxl-builtins.md
            ├── gxl-examples.md
            └── patch-file.md
```

## Notes

- Prefer installing the whole collection when you want routing from `gx-skills` to the nested skills.
- When the skill text conflicts with the target repo, `src/` and tests in `galaxy-flow` remain the source of truth.
