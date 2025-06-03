# OpenHands 项目分析与部署指南

本文档概述了 OpenHands 项目的整体架构、目录结构及二次开发要点，并给出经过验证的本地部署流程。

## 项目概览

OpenHands 是一个基于 Python 的 AI Agent 框架，提供了丰富的运行时环境与前端交互界面，支持插件式扩展和自定义微代理（Microagents）。项目采用 `poetry` 管理依赖，提供 Docker 镜像以便快速启动。

核心目录结构如下：

- **openhands/**：核心代码，包含 Agent 实现、控制器、事件流、运行时等模块。
- **frontend/**：React 编写的 Web 前端，提供用户交互界面。
- **microagents/**：可复用的 Microagent 文档，帮助代理执行特定任务。
- **evaluation/**：基准测试与评测脚本。
- **tests/**：单元测试与运行时集成测试，用于验证功能。

更多架构细节可参考 `openhands/README.md` 中的说明，其中包含各组件之间的关系及事件流图示【F:openhands/README.md†L1-L38】。

## 二次开发要点

1. **配置管理**：
   - 配置文件示例位于 `config.template.toml`，实际运行时可复制为 `config.toml` 并按需修改。
   - `openhands/core/config` 中的 `load_app_config()` 会从 TOML、环境变量及命令行参数加载配置，支持递归处理并保持类型安全【F:openhands/core/config/README.md†L1-L54】。

2. **扩展 Agent**：
   - 新增 Agent 实现可放置在 `openhands/agenthub` 中，评测相关脚本位于 `evaluation` 目录。
   - Microagent 机制可通过在 `microagents/` 添加 markdown 文件实现，或在具体仓库的 `.openhands/microagents/` 提供私有指令【F:microagents/README.md†L1-L32】。

3. **前端开发**：
   - 前端代码位于 `frontend/`，使用常见的 npm/yarn 工作流。开发前端需确保 Node.js 环境。

4. **测试**：
   - 运行所有单元测试：`poetry run pytest ./tests/unit`。
   - 运行运行时集成测试：`poetry run pytest ./tests/runtime`【F:tests/unit/README.md†L1-L8】【F:tests/runtime/README.md†L9-L25】。

## 本地部署流程（Docker）

以下步骤在 Ubuntu 22.04 环境验证通过：

1. **安装 Docker**
   ```bash
   curl -fsSL https://get.docker.com | sh
   sudo usermod -aG docker $USER
   ```
   注：安装后需重新登录以生效。

2. **拉取并运行 OpenHands 镜像**
   ```bash
   docker pull docker.all-hands.dev/all-hands-ai/runtime:0.39-nikolaik
   docker run -it --rm --pull=always \
       -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:0.39-nikolaik \
       -e LOG_ALL_EVENTS=true \
       -v /var/run/docker.sock:/var/run/docker.sock \
       -v ~/.openhands-state:/.openhands-state \
       -p 3000:3000 \
       --add-host host.docker.internal:host-gateway \
       --name openhands-app \
       docker.all-hands.dev/all-hands-ai/openhands:0.39
   ```
   启动完成后，可在浏览器访问 `http://localhost:3000`。

3. **首次启动配置**
   - 在界面中选择 `LLM Provider` 与 `LLM Model`，并填写对应的 API Key。
   - 若模型列表中不存在所需模型，可在 `Settings -> LLM -> Advanced` 中手动填写模型名称及 Base URL。

4. **运行测试**（可选）
   ```bash
   poetry install
   poetry run pytest ./tests/unit -q
   ```
   确认测试全部通过后即可开始使用。

## 结论

通过以上步骤，可在本地快速部署并体验 OpenHands。结合本文档提供的目录结构和扩展要点，开发者可以方便地进行二次开发或集成更多功能。

