# 🤖 基于 n8n 与 DeepSeek 的 AI 智能效率助理 Agent

## 📖 项目简介
本项目基于开源工作流引擎 **n8n**，结合 **DeepSeek 大语言模型** 与 **飞书开放平台 API**，构建了一个具备多源数据聚合与自主决策能力的个人效率助理 Agent。
系统实现了从底层容器化部署、大模型提示词工程、复杂第三方 API 集成，到 UI 卡片级消息推送的端到端完整闭环。

## 🏗️ 工作流架构展示

- **触发层**：`Schedule Trigger` (定时触发，每日早晨8点自动运行)
- **决策层**：`AI Agent` + `DeepSeek Chat Model`
- **工具层 (Tools)**：挂载了两个自定义工具
  - 飞书日历 API (`Calendar:listEvents`)
  - 飞书多维表格 API (`Bitable:table:record:search`)
- **展示层**：飞书卡片 2.0 交互式消息 JSON 渲染（通过 HTTP Request 节点触发 Webhook）

> *注：完整的节点连线与参数配置，请下载仓库内的 `workflow.json` 导入 n8n 查看。*

## 💡 核心技术亮点与难点攻克

1. **底层容器化与网络优化**
   - 基于 `Windows + WSL 2` 构建 Linux 底座，利用 Docker 容器化部署 n8n。
   - 配置国内镜像加速源，解决网络拉取超时问题，并挂载 `Volume` 保证工作流数据持久化。

2. **Agent 工作流与工具调用 (Function Calling)**
   - 采用 `AI Agent + Tool` 架构，设定严谨的 System Prompt 约束输出格式。
   - 实现了大模型根据用户意图自主生成动态参数，并调用对应的飞书 API 工具。

3. **复杂鉴权与 API 深度对接**
   - 深度攻克 OAuth 2.0 双重身份鉴权（`tenant_access_token` 应用身份 vs `user_access_token` 用户身份）。
   - 解决了飞书开放平台常见的 `403 Forbidden` 及 `99991679` 权限拦截问题。

4. **AI辅助 JavaScript 数据清洗算法**
   - **时间戳处理**：ai辅助算法 `Math.floor((Math.floor(Date.now()/1000) + 28800) / 86400) * 86400 - 28800`，完美解决 UTC 与东八区（UTC+8）的偏差，精准计算出“今天 0 点”的 Unix 时间戳。
   - **数据剥离**：使用 `JSON.stringify($json.output).slice(1, -1)` 清除 LLM 输出的多余转义字符，确保适配飞书 Markdown 卡片渲染规范。

## 🔧 如何运行此项目？
1. 确保本地已安装 Docker 环境。
2. 运行以下命令启动 n8n：
   ```bash
   docker run -d --name n8n --restart unless-stopped -p 5678:5678 -e GENERIC_TIMEZONE="Asia/Shanghai" -e TZ="Asia/Shanghai" -e N8N_SECURE_COOKIE=false -v n8n_data:/home/node/.n8n docker.1panel.live/n8nio/n8n:latest
