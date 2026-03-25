# 开发文档（本地开发）

## 环境准备

1. 安装 **Python 3.12+** 与 **uv**（https://docs.astral.sh/uv/）。
2. 克隆仓库后在项目根目录执行：

```bash
uv sync
```

## 运行核心（后端 + 内置打包的 WebUI）

```bash
uv run main.py
```

- 默认暴露 **http://localhost:6185**（见仓库 `AGENTS.md`）。
- 首次运行会确保 `data/`、插件目录、临时目录等存在（见 `main.py` 中 `check_env`）。

## 单独开发 Dashboard（前端热更新）

```bash
cd dashboard
pnpm install   # 未安装 pnpm 时：npm install -g pnpm
pnpm dev
```

- 默认 **http://localhost:3000**。
- 修改 Vue 源码时保持组件化，避免重复逻辑（仓库约定）。

## 代码规范（提交前）

在项目根目录：

```bash
ruff format .
ruff check .
```

- 新注释使用 **英文**（仓库约定）。
- 路径处理优先 **`pathlib.Path`**，数据/临时目录优先使用 **`astrbot.core.utils.path_utils`**（见 `AGENTS.md`）。

## 阅读代码的推荐入口

| 目标 | 建议文件 |
|------|-----------|
| 启动流程 | `main.py`, `astrbot/core/initial_loader.py`, `astrbot/core/core_lifecycle.py` |
| 消息从进入到回复 | `astrbot/core/pipeline/scheduler.py`, `stage_order.py`, `process_stage/stage.py` |
| 内置 Agent 循环 | `astrbot/core/agent/runners/tool_loop_agent_runner.py`, `astrbot/core/astr_agent_run_util.py` |
| 主 Agent 工具与 Skills | `astrbot/core/astr_main_agent.py`, `astrbot/core/skills/skill_manager.py` |
| 插件扩展 | `astrbot/core/star/`, `astrbot/core/star/star_manager.py` |
| Web API | `astrbot/dashboard/server.py`, `astrbot/dashboard/routes/` |
| 官方插件开发说明 | `docs/zh/dev/star/`、`docs/zh/use/plugin.md` |

## 测试

```bash
uv run pytest
```

（具体范围以 `tests/` 与项目配置为准。）
