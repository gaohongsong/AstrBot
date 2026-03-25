# 技术栈（依赖、框架与组件）

## 运行时与语言

- **Python**：`pyproject.toml` 声明 `requires-python = ">=3.12"`（`main.py` 仍保留对 3.10+ 的检查文案，以仓库实际约束为准）。
- **包管理 / 运行**：推荐 **uv**（`uv sync`、`uv run main.py`）；CLI 入口 `astrbot` 映射到 `astrbot.cli.__main__:cli`。
- **构建**：**Hatchling**，自定义 `**scripts/hatch_build.py`** 在打包时构建 Dashboard 并打入 `astrbot/dashboard/dist`。

## 后端核心依赖（节选）

以下为 `pyproject.toml` 中与架构强相关的部分，完整列表以文件为准。


| 类别           | 代表依赖                                                                                                                 | 用途                         |
| ------------ | -------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| Web 框架       | `quart`, `hypercorn`（在 dashboard 中引用）                                                                                | 异步 HTTP / WebUI API        |
| 数据与 ORM      | `sqlalchemy[asyncio]`, `sqlmodel`, `aiosqlite`                                                                       | 异步数据库访问                    |
| LLM / API    | `openai`, `anthropic`, `google-genai`, `dashscope` 等                                                                 | 多厂商模型与语音相关                 |
| Agent / 工具协议 | `mcp`                                                                                                                | Model Context Protocol 客户端 |
| IM / 平台 SDK  | `python-telegram-bot`, `slack-sdk`, `py-cord`, `lark-oapi`, `dingtalk-stream`, `aiocqhttp`, `qq-botpy`, `wechatpy` 等 | 各平台适配                      |
| 任务调度         | `apscheduler`                                                                                                        | 定时任务（与 Cron 管理器配合）         |
| 沙盒 / 容器      | `aiodocker`, `shipyard-python-sdk`, `shipyard-neo-sdk`                                                               | Agent 沙盒与 Shipyard 集成      |
| 知识检索         | `faiss-cpu`, `rank-bm25`, `jieba`                                                                                    | 向量与文本检索组件                  |
| 文档与媒体        | `pypdf`, `markitdown-no-magika`, `pillow`, `pydub`                                                                   | 文件解析与多媒体                   |
| 其他           | `pydantic`, `loguru`, `tenacity`, `websockets`, `watchfiles`                                                         | 配置校验、日志、重试、热更新等            |


## 前端（Dashboard）

路径：`dashboard/package.json`。

- **Vue 3** + **Vite** + **TypeScript**
- **Vuetify 3**、**Pinia**、**Vue Router**、**vue-i18n**
- 编辑器与展示：**Monaco**、**markdown-it**、**mermaid**、**katex** / **shiki** 等

## 代码质量

- **Ruff**：format + lint（`[tool.ruff]`）
- **Pyright**：basic 模式（`[tool.pyright]`）
- 测试：**pytest**、**pytest-asyncio**（`dependency-groups.dev`）

## 默认服务端口（常见）

- **6185**：AstrBot WebUI / API（`AGENTS.md` 与 `compose.yml` 注释）
- **6199**：QQ 个人号 WebSocket（可选，见 `compose.yml`）
- **Dashboard 独立开发**：Vite 默认 **3000**（`pnpm dev`）

