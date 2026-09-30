# Gx Skills

Focused skills for `gx` (galaxy-flow) — the GXL workflow engine and CLI.

The repo is organized as a top-level router skill plus nested tool-specific skills under `skills/`. gops (galaxy-ops) skills live separately in [gops-skills](https://github.com/galaxio-labs/gops-skills).

## Available Skills

- `gx-skills` — top-level router for gx tasks.
- `gx-engineering` — recommended `gx` usage: `run`/`adm`/`init`/`mod`/`doc`/`check`/`self`, GXL authoring & pitfalls (`gx.shell`/`gx.cmd`, `silence`, backgrounding), built-in `gx.*` capabilities, and `_gal/` conventions.

## Installation

### With `gops` (recommended)

If you have `gops` installed, use its native installer — no shell script, no `python3` / `ruby`:

```bash
gops self skill install --source galaxio-labs/gx-skills
gops self skill list --source galaxio-labs/gx-skills
```

### With `install.sh`

Use this when you do not have `gops` yet (bootstrapping). Install the whole collection (router + nested skills):

```bash
./install.sh                          # install all available platforms
./install.sh --zed                    # only Zed (~/.agents/skills/)
./install.sh --dir ~/my/skills        # custom directory
```

Install a single skill by name (local checkout first, remote clone fallback):

```bash
./install.sh gx-engineering --codex
./install.sh gx-engineering --claude
```

Remote install (no local checkout):

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/galaxio-labs/gx-skills/main/install.sh) gx-engineering
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
    └── gx-engineering/
        └── SKILL.md
```

## Notes

- Prefer installing the whole collection when you want routing from `gx-skills` to the nested skills.
- When the skill text conflicts with the target repo, `src/` and tests in `galaxy-flow` remain the source of truth.
