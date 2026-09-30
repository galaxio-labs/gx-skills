# 变更日志

## [0.3.0] - 2026-09-30

### 拆分：`gx-engineering` → `gx-cli` + `gxl-authoring`

- **`gx-cli`**：CLI 运维 —— `gx run`/`adm`/`init`/`mod`/`doc`/`check`/`self`/`self skill`、`_gal` 布局、flags、项目初始化、自升级、与 gops 的集成与陷阱
- **`gxl-authoring`**：GXL 编写 —— 结构（`mod`/`env`/`flow`/`fn`/`activity`）、flow 头部、`#[...]` 注解、控制流、变量与内置常量、内置 `gx.*` 能力、`gx.patch_file`、示例、坑位；深度材料在 `references/`（原 `gx-engineering/references/*` 迁移过来）
- 顶层路由 `SKILL.md` 与 README 同步；`gx-engineering` 名称移除

## [0.2.0] - 2026-09-30

### gx-engineering：基于 `galaxy-flow/docs` 补齐 GXL 参考

- `gx-engineering/SKILL.md` 重写扩充：命令（含 `gx run --exists`、`gx self skill`）、flags、`_gal` 约定、GXL 结构（`mod`/`env`/`flow`/`fn`/`activity`）、flow 头部三种形式、注解、调用语法、控制流、变量（大小写不敏感、`ENV_` 前缀）、内置常量
- 新增 `skills/gx-engineering/references/`（自 `galaxy-flow/docs` 整理）：
  - `gxl-syntax.md`：语法骨架（EBNF）与结构 / 注解 / 控制流 / 变量 / 内置常量
  - `gxl-builtins.md`：全部 `gx.*` 内置能力与表达式函数（用途、参数、示例、约束）
  - `gxl-examples.md`：示例集（assert / dryrun / fun / read / shell / template / transaction / vars）
  - `patch-file.md`：`gx.patch_file` 的 marker 模型、参数、MVC 设计与约束
- 顶层路由 `SKILL.md` 与 README 同步

## [0.1.1] - 2026-09-30

### 文档

- README 安装说明改为 **With `gx`（推荐）**：用 `gx self skill install`（需 `gx >= 0.16.0`），无需 `gops`；`gops self skill install --source galaxio-labs/gx-skills`（需 `gops >= 2.0.13`）降为备选；`install.sh` 保留为两者都没有时的引导安装

## [0.1.0] - 2026-09-30

### 初始版本

- 从 `gops-skills` 拆出 `gx`（galaxy-flow）相关 skill，独立成仓
- 顶层路由 skill `gx-skills`，按任务路由到 `gx-engineering`
- `gx-engineering`：`gx run/adm/init/mod/doc/check/self`、GXL 编写（`gx.shell`/`gx.cmd`、`silence`、后台化）、内置 `gx.*` 能力、`_gal/` 约定与常见陷阱
- 附 `install.sh`（按 skill 安装 / 整包安装，本地源优先、远程 clone 兜底）与 `agents/openai.yaml` 元数据
