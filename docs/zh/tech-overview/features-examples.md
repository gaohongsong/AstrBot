# 产品核心功能与样例

以下功能均可在官方文档与 WebUI 中配置；此处从**产品能力**角度归纳，并给出**可操作的典型路径**（非唯一方式）。

## 1. 多平台 IM 接入

- **能力**：同一套机器人逻辑对接 QQ（含 OneBot/NapCat 等）、Telegram、企业微信、飞书、钉钉、微信公众号、Slack、Discord、LINE、Satori、Misskey 等（详见根目录 `README.md` 表格）。
- **样例**：在 WebUI「平台」中添加对应适配器，按各平台文档填写 Token / Webhook / 长轮询等；Docker 部署时可按需暴露 `6199`、`6195`、`6196` 等端口（见 `compose.yml` 注释）。

## 2. 大模型对话与人格（Persona）

- **能力**：多 Provider、唤醒词、会话级开关、人格与系统提示配置。
- **样例**：「模型提供商」添加 OpenAI 兼容或官方渠道 → 「人格」绑定模型与提示词 → 在群中 `@` 机器人或使用唤醒前缀触发（由 Pipeline 中 WakingCheck / ProcessStage 逻辑决定）。

## 3. 内置 Agent（工具循环）

- **能力**：在主 Agent 上挂载浏览器、Shell/Python（受权限与执行环境影响）、文件上下传、知识库查询、MCP 工具、子 Agent Handoff、Cron 工具等（见 `astr_main_agent.py` 中工具导入与装配）。
- **样例**：用户：「查一下某文档里的配置项并总结」→ `ToolLoopAgentRunner` 多轮调用检索/浏览器/知识库工具 → 流式或完整回复到 IM。

## 4. 第三方 Agent 执行器

- **能力**：将「思考 + 做事」交给 Dify / Coze / 百炼应用 / DeerFlow 等平台（见 `docs/zh/use/agent-runner.md`）。
- **样例**：新增「Agent 执行器」类型提供商 → 在「配置 → Agent 执行方式」中切换默认执行器。

## 5. 插件市场（Star）

- **能力**：社区插件一键安装，扩展指令、游戏、管理功能等。
- **样例**：WebUI「插件」搜索安装；对话中 `/plugin` 查看列表（见 `docs/zh/use/plugin.md`）。

## 6. Skills（Anthropic Skills 风格）

- **能力**：ZIP 上传技能包，目录名即技能 ID，根目录含 `SKILL.md`；系统提示中仅注入名称与描述，由模型按需读取全文（渐进式披露），见 `docs/zh/use/skills.md` 与 `SkillManager`。
- **样例**：上传 `my-skill.zip` → 解压为 `my-skill/SKILL.md` → 启用沙盒或 Local 执行环境 → 用户自然语言任务匹配该技能描述时，Agent 按 `SKILL.md` 指引执行。

## 7. 知识库（RAG）

- **能力**：文档切块、向量与检索（FAISS 等），可选 Agent 主动查询模式（`kb_agentic_mode` 等配置在 `astr_main_agent.py` 的 `MainAgentBuildConfig` 中可见）。

## 8. MCP

- **能力**：通过 MCP 将外部服务以工具形式暴露给 Agent（见 `docs/en/use/mcp.md` / 中文站对应页面）。

## 9. Agent 沙盒

- **能力**：隔离执行 Python/Shell 与 Skills 相关操作，降低对宿主机的风险；可与 Shipyard 组合部署（见 `docs/zh/use/` 下沙盒文档与 `compose-with-shipyard.yml`）。

## 10. Web 聊天与国际化

- **能力**：内置 Web Chat UI；Dashboard 与产品层面支持 i18n（前端 `vue-i18n`）。

---

**小结**：AstrBot 的「默认体验」是 **IM 进线 → Pipeline 校验 → ProcessStage 选插件或主 Agent → Provider 补全与工具循环 → 装饰与回发**；扩展上 **Star 改行为、Skills 教模型怎么做、MCP 接外部系统**。
