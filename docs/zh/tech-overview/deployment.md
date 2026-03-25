# 部署架构与方法

## 部署架构（逻辑）

典型生产形态为**单容器（或单进程）+ 持久化数据卷**：

- **AstrBot 主服务**：Quart/Hypercorn 提供 WebUI 与 API（默认 **6185**）；内嵌 Pipeline、Provider、插件、内置 Agent。
- **数据卷**：将宿主目录挂载到容器内 **`/AstrBot/data`**（或等价路径），包含配置、数据库、插件、知识库、Skills、可选的 `dist` 等（见 `compose.yml` 与 `get_astrbot_data_path` 相关逻辑）。
- **可选组件**：
  - **NapCat** 等：与 QQ 协议对接时，可使用社区提供的组合 Compose（`compose.yml` 顶部注释链接）。
  - **Shipyard 沙盒**：需要隔离执行能力时，使用 `compose-with-shipyard.yml`，并配置 Shipyard 访问凭证（详见官方「Agent 沙盒」文档）。

```text
                    ┌─────────────────────────────┐
  Users (IM/Web) ──►│ AstrBot (6185, optional WS) │
                    │  Pipeline / Agent / Stars    │
                    └──────────────┬──────────────┘
                                   │ volume
                              ./data (config, db, plugins, kb, skills)
                    ┌──────────────▼──────────────┐
                    │ Optional: Shipyard Sandbox │
                    └─────────────────────────────┘
```

## 常用部署方法（摘要）

详细步骤以官方文档为准，此处仅作索引式说明。

| 方式 | 适用场景 | 参考 |
|------|-----------|------|
| **uv / pip 工具** | 本机或自有服务器快速运行 | 根目录 `README.md`：`uv tool install astrbot`、`astrbot run` |
| **Docker / Compose** | 标准化、易迁移 | `compose.yml`、`docs/zh/deploy/astrbot/docker.md` |
| **Compose + Shipyard** | 需要沙盒执行 | `compose-with-shipyard.yml`、沙盒文档 |
| **面板 / 云镜像** | 宝塔、1Panel、CasaOS、K8s 等 | `docs/zh/deploy/astrbot/`、`docs/en/deploy/` 下各文 |
| **桌面端** | 偏 ChatUI 的个人使用 | AstrBot-desktop、Launcher（见 README，非服务器首选） |

## Docker 最小示例

```bash
git clone https://github.com/AstrBotDevs/AstrBot
cd AstrBot
docker compose up -d
```

- 端口：**6185** 必选；**6199** 等按平台按需开启。
- 大陆网络拉镜像困难时，可按 `docker.md` 使用镜像加速地址。

## 运维注意点

- **时区**：Compose 示例中常挂载 `/etc/localtime` 或设置 `TZ`，避免 Cron 与日志时间偏差。
- **密钥**：模型 API Key、平台 Token、Shipyard 凭证等应通过环境变量或安全配置注入，避免写入公开仓库。
- **升级**：关注 `changelogs/` 与主版本迁移说明；`uv tool upgrade astrbot` 或拉取新镜像。
