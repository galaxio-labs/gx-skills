# 变更日志

## [0.1.1] - 2026-09-30

### 文档

- README 安装说明改为 **With `gx`（推荐）**：用 `gx self skill install`（需 `gx >= 0.16.0`），无需 `gops`；`gops self skill install --source galaxio-labs/gx-skills`（需 `gops >= 2.0.13`）降为备选；`install.sh` 保留为两者都没有时的引导安装

## [0.1.0] - 2026-09-30

### 初始版本

- 从 `gops-skills` 拆出 `gx`（galaxy-flow）相关 skill，独立成仓
- 顶层路由 skill `gx-skills`，按任务路由到 `gx-engineering`
- `gx-engineering`：`gx run/adm/init/mod/doc/check/self`、GXL 编写（`gx.shell`/`gx.cmd`、`silence`、后台化）、内置 `gx.*` 能力、`_gal/` 约定与常见陷阱
- 附 `install.sh`（按 skill 安装 / 整包安装，本地源优先、远程 clone 兜底）与 `agents/openai.yaml` 元数据
