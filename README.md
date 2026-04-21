# Atlas Zero Agent

`Atlas Zero Agent` 是一个面向通用对话场景的 `chatbox agent` 项目。

它以聊天作为统一入口，内部通过多 agent 协作完成规划、执行、知识检索、记忆管理、安全审查和资源调度，目标是做成一个可扩展、可服务化、可观测的 agent runtime，而不是一个只会单轮回答的聊天壳。

## Vision

- 一个对外统一的聊天入口
- 一个对内可编排的 agent runtime
- 一套可持续扩展的 memory、RAG 与 tools 体系
- 一套适合产品化落地的 API、状态管理与治理能力

## Core Ideas

- `Chatbox First`: 用户只面对一个自然的聊天入口
- `Agent Runtime Inside`: 内部通过多个职责明确的 agents 协同工作
- `FastAPI Native`: 后端优先采用服务化接口设计
- `LangChain + LangGraph`: 用组件抽象连接模型、工具和知识，用状态图管理多步执行
- `Observable by Default`: 从一开始就考虑 trace、审计、成本和运行状态

## Agent Roles

项目借用了《海贼王》蛋头岛 `Vegapunk satellites` 的命名体系，但这些名字不只是皮肤，而是对应真实职责：

- `Stella`: 主控入口与 orchestrator
- `Atlas`: 执行型 agent，负责工具调用和任务落地
- `Edison`: 规划型 agent，负责步骤拆解与工具编排
- `Pythagoras`: 知识与记忆 agent，负责 RAG、总结与长期上下文
- `Shaka`: 安全与规则 agent，负责 guardrails、风险判定与权限控制
- `Lilith`: 探索型 agent，负责多路搜索、试探和实验性路径
- `York`: 资源型 agent，负责模型路由、配额、队列和成本控制

## Architecture

建议采用分层结构：

- `Interface Layer`: `FastAPI` 对外接口、流式输出、会话 API
- `Application Layer`: 请求编排、权限、会话上下文、服务层
- `Agent Runtime Layer`: `LangGraph` 编排多 agent 状态流转
- `Knowledge Layer`: session memory、long-term memory、RAG
- `Tool Layer`: 本地工具、外部 API、未来的 `MCP` 接入
- `Observability Layer`: trace、审计、指标、成本治理

更详细的工程版设计见：

- `atlas-zero-agent-design.md`
- `docs/architecture.md`

## Planned Stack

- `FastAPI`
- `Pydantic`
- `LangChain`
- `LangGraph`
- `PostgreSQL`
- `Redis`
- `pgvector`
- `LangSmith` or `OpenTelemetry`

## API Direction

第一阶段建议优先做这些接口：

- `POST /api/chat`
- `POST /api/chat/stream`
- `GET /api/sessions/{session_id}`
- `POST /api/sessions/{session_id}/messages`
- `GET /api/agents`
- `GET /api/health`

## Roadmap

### Phase 1

- 基础 chat API
- 流式输出
- session 持久化
- 最小多 agent 执行链路
- 基础 memory 与 RAG

### Phase 2

- 多 agent 扩展
- 用户长期记忆
- 文件上传与知识入库
- 模型路由与预算控制
- 更完整的 observability

### Phase 3

- 异步任务与队列
- 人工审批
- `MCP` 工具接入
- 多租户与插件化

## Current Status

当前仓库处于设计和骨架规划阶段，重点先明确：

- 命名体系
- agent 职责边界
- runtime 架构
- API 设计
- 最小可运行实现路径

## Repository Notes

- 设计草案：`atlas-zero-agent-design.md`
- 工程架构：`docs/architecture.md`

后续可以从一个最小闭环开始：`Stella -> Shaka -> Edison -> Pythagoras -> Atlas`。
