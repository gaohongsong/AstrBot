# 扩展与对比说明（FAQ）

## 1. AstrBot 如何支持 Skills？

- **规范**：遵循 **Anthropic / OpenAI Codex CLI 常见的 Skills 形态**——每个技能是一个文件夹，内含 **`SKILL.md`**（可选 YAML frontmatter 中的 `name`、`description`）。见 `astrbot/core/skills/skill_manager.py` 中 `_parse_frontmatter_description`、`build_skills_prompt`。
- **存储**：技能安装在数据目录下的 **`skills/`**（`get_astrbot_skills_path()`）。
- **管理**：WebUI「插件」页中的 **Skills** 入口；API 实现在 `astrbot/dashboard/routes/skills.py`（上传 ZIP、列表、同步沙盒等）。
- **运行方式**：Skills 往往包含需在沙盒或本机执行的说明与脚本，因此依赖 **执行环境**：
  - **Sandbox**：推荐，需启用 AstrBot Agent 沙盒（及 Shipyard 等部署方式）。
  - **Local**：在 AstrBot 进程所在环境执行，**安全风险高**；且官方文档说明仅 **管理员** 请求才允许真正操作本地环境（见 `docs/zh/use/skills.md`）。
- **上下文策略**：**渐进式披露**——初始只把技能名称与简短描述注入系统提示；由模型在需要时读取完整 `SKILL.md`，节省 Token（`build_skills_prompt` 的设计意图）。

## 2. Agent 框架是什么？

在 **AstrBot 内置路径**下，Agent 不是单独引入的第三方「Agent 框架名」，而是自研组合：

- **循环引擎**：**`ToolLoopAgentRunner`**（`tool_loop_agent_runner.py`）继承 `BaseAgentRunner`，实现「模型输出 → 解析工具调用 → 执行 → 写回消息 → 再请求模型」的闭环。
- **上下文与压缩**：同目录下的 `context/`（`ContextManager`、`ContextCompressor`、`TokenCounter` 等）。
- **工具执行**：`FunctionToolExecutor` / `astrbot/core/astr_agent_tool_exec.py` 等与主 Agent 装配衔接。
- **主 Agent 编排**：**`astr_main_agent.py`** 将 MCP、Skills、知识库、沙盒/本地工具、子 Agent、Cron 等组装为 **`ToolSet`** 交给上述 Runner。

若配置为 **第三方 Agent 执行器**，则「循环」主要在 **Dify / Coze / 百炼 / DeerFlow** 侧完成，AstrBot 通过 **`ThirdPartyAgentSubStage`** 转发，详见 `docs/zh/use/agent-runner.md`。

## 3. 它和 OpenClaw 的区别？

需要先澄清 **OpenClaw 在仓库中的含义**：

- **腾讯相关接口/入口**：文档与代码里出现的 **OpenClaw** 多指 **官方开放平台路径或产品名的一部分**，例如：
  - 个人微信适配器基于腾讯 **`openclaw-weixin`** 接口（见 `docs/zh/platform/weixin_oc.md`）。
  - QQ 开放平台文档中的链接路径包含 **`openclaw`**（见 `docs/zh/platform/qqofficial/websockets.md`）。
- **这些并不是**与 AstrBot 对标的「另一个开源 Agent 框架」；而是 **IM 平台侧的能力或品牌命名**。

若你指的是**其它**也叫 OpenClaw 的独立产品（例如某 Agent 运行时），本仓库未提供对比材料；建议用下列维度自行对照：

| 维度 | AstrBot |
|------|---------|
| 主场景 | 多 IM 平台聊天机器人 + Web 管理面板 |
| Agent | 内置 Tool Loop + 可选第三方 Agent 平台 |
| 扩展 | Star 插件、MCP、Skills、知识库 |
| 执行安全 | 可选沙盒、Local 受限 |

## 4. SRE 场景中可以用吗？

**可以，但是「运维机器人 / 协作入口」型用法，而不是替代专业 APM/日志平台。**

适合的方向包括：

- **告警与协作渠道**：对接 **Slack / 飞书 / 钉钉 / Telegram** 等，将摘要或 Runbook 查询入口放到对话里。
- **Runbook + RAG**：知识库挂载运维文档，Agent 按权限查询与总结（注意敏感信息与幻觉风险）。
- **定时与自动化**：内置 **Cron 工具**、APScheduler，可做轻量定时任务触发（复杂编排仍建议专业调度系统）。
- **受控执行**：在**沙盒**中执行只读脚本或隔离命令，避免在生产宿主机上直接 `Local` 执行未审计代码。

不适用或需谨慎的场景：关键生产变更的**唯一控制面**、无审计的任意 Shell、强合规下的凭证处理等——需额外网关、审批流与最小权限设计。

## 5. 插件（Star）和 Skill 有什么区别？「个人工时记录」建议用哪种？

| 对比项 | Star（插件） | Skill |
|--------|----------------|--------|
| 本质 | Python 包，注册 Handler，**确定性代码路径** | `SKILL.md` + 资源，**教模型如何完成任务**，常配合代码执行 |
| 触发 | 指令、正则、事件类型等明确规则 | 依赖模型理解描述后是否「选用」该技能 |
| 状态与存储 | 易于用 SQLite/文件/API 做**强一致业务数据** | 适合流程与模板，持久化仍可通过工具/脚本但边界更「软」 |
| 分发 | 插件市场 ZIP / 本地开发 | ZIP 上传到 Skills |
| 升级与测试 | 单元测试、版本号清晰 | 需验证模型是否稳定遵守 `SKILL.md` |

**「个人工时记录」建议**：

- **更推荐 Star（插件）**：若你需要 **固定命令**（如 `/工时 start`、`/工时 stop`）、**可靠写入数据库或表格**、**按用户/会话隔离统计**、**不依赖模型每次理解自然语言**，插件最稳。
- **可选补充 Skill**：若你希望用**自然语言**（「刚才开会两小时帮我记到项目 A」）由 Agent **理解意图再调用已有工具**，可以写一个 Skill 描述流程，并**让 Skill 中的步骤调用插件注册的底层能力或 HTTP API**；但**单一事实来源**仍建议放在插件或后端服务里，而不是完全交给自由文本。

**不推荐**仅用 Skill、且无沙盒/无管理员限制地在 Local 环境执行任意统计脚本——安全与一致性都难保证。

---

以上解读基于当前仓库源码与官方文档；具体行为以你所部署的版本配置为准。
