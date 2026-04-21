# Architecture

## Overview

`Atlas Zero Agent` 采用“单 chatbox 入口 + 多 agent 内部协作”的架构。

对外，系统暴露为一个统一的聊天服务；对内，系统通过 `Stella` orchestrator 把请求分发给不同职责的 agent 节点，结合 `LangGraph` 管理状态流转，结合 `LangChain` 管理模型、工具、检索器和提示词。

工程目标有三点：

- 让 chat 成为统一入口
- 让 agent runtime 成为核心执行层
- 让 memory、tools、observability 从一开始就是一等公民

## Design Goals

- 支持通用聊天与多轮对话
- 支持工具调用与多步任务执行
- 支持短期记忆、长期记忆与知识检索
- 支持流式输出和可恢复执行
- 支持成本、权限、审计和 trace 管理
- 支持后续扩展到多租户和插件体系

## High-Level Architecture

```text
Client / UI
  -> FastAPI API Layer
  -> Application Services
  -> Stella Orchestrator
  -> LangGraph Runtime
     -> Shaka Guard
     -> Edison Planner
     -> Pythagoras Memory / RAG
     -> Atlas Executor
     -> Lilith Explorer
     -> York Resource Manager
  -> Tool Layer / Memory Layer / Model Layer
  -> Persistence + Observability
```

## Layered Architecture

### 1. Interface Layer

职责：

- 对外暴露 HTTP API
- 提供同步与流式聊天接口
- 提供会话查询与管理接口
- 提供健康检查、调试和管理接口

建议技术：

- `FastAPI`
- `Pydantic`
- `SSE`，后续可选 `WebSocket`

建议目录：

- `app/api/routes/`
- `app/api/schemas/`

这一层只处理：

- 请求解析
- 参数校验
- 响应序列化
- 状态码和错误格式

这一层不应该直接写复杂 agent 逻辑。

### 2. Application Layer

职责：

- 接收 API 请求并转成应用层命令
- 维护 session、user、tenant 上下文
- 管理鉴权、限流、权限边界
- 调用 orchestrator 与服务层

建议目录：

- `app/services/`
- `app/core/`

应用层负责解决“系统怎么接住一个请求”，而不是“模型怎么思考”。

### 3. Agent Runtime Layer

职责：

- 管理多 agent 状态流转
- 管理节点输入输出和共享状态
- 管理 checkpoint、中断、恢复和回放
- 统一 tool call、fallback 和 error routing

建议技术：

- `LangGraph`
- `LangChain`

建议目录：

- `app/runtime/`
- `app/agents/`

核心设计原则：

- 每个 agent 有清晰职责
- 节点之间通过统一 `state` 共享信息
- 最终只允许少数节点产生对用户可见的最终输出
- 失败路径必须可观测、可恢复、可降级

### 4. Knowledge and Memory Layer

职责：

- 保存当前会话消息
- 保存长期用户偏好与任务历史
- 提供知识库检索与重排
- 提供对话摘要和记忆压缩

建议拆成三类存储：

- `Session Store`
- `Memory Store`
- `Vector Store`

建议目录：

- `app/memory/`

建议原则：

- 短期记忆和长期记忆分开建模
- 检索上下文和用户消息分开处理
- 避免把所有历史直接塞进 prompt
- 优先通过摘要、选择性检索和结构化 memory 控制上下文长度

### 5. Tool and Integration Layer

职责：

- 封装内部工具
- 封装第三方 API
- 控制工具权限与调用边界
- 统一工具描述、输入和输出格式

建议目录：

- `app/tools/`

工具层应该满足：

- 可测试
- 可复用
- 可审计
- 可在 runtime 中统一注册

后续可预留：

- `MCP` 接入
- browser / search 工具
- filesystem 工具
- knowledge connectors

### 6. Observability and Governance Layer

职责：

- 跟踪每次请求的执行链路
- 记录 tool call、模型调用与成本
- 记录安全拦截与人工审批事件
- 输出 metrics、audit log 和 trace

建议目录：

- `app/observability/`

建议技术：

- `LangSmith`
- `OpenTelemetry`
- `Prometheus`
- `Grafana`

这个层不是后补件，而是 agent 系统能否产品化的关键部分。

## Agent Topology

### External Identity

- `Atlas Zero`: 对外统一产品身份

### Internal Roles

- `Stella`: orchestrator / router
- `Shaka`: safety and policy guard
- `Edison`: planner and workflow builder
- `Pythagoras`: memory and retrieval manager
- `Atlas`: executor and tool runner
- `Lilith`: exploration and fallback agent
- `York`: resource and budget manager

## Minimum Execution Flow

第一阶段建议实现如下链路：

```text
User Message
  -> API Layer
  -> Session Service
  -> Stella
  -> Shaka
  -> Edison
  -> Pythagoras
  -> Atlas
  -> Stella Response Composer
  -> Streaming / Final Response
```

各节点职责：

- `Stella`: 创建运行上下文，决定是否直接回答还是进入执行链路
- `Shaka`: 校验风险、权限和策略边界
- `Edison`: 生成任务计划、决定工具和步骤
- `Pythagoras`: 读取 session memory、long-term memory 和知识库
- `Atlas`: 执行工具调用、收集结果、产出 observation
- `Stella Response Composer`: 汇总结果并形成最终回答

## Optional Extended Flow

在第二阶段可扩展：

```text
Stella
  -> Shaka
  -> Edison
  -> Pythagoras
  -> Lilith
  -> Atlas
  -> York
  -> Stella
```

扩展后的作用：

- `Lilith`: 做多路探索、候选方案试探、失败路径补救
- `York`: 做模型选择、预算控制、异步任务排队和优先级管理

## Runtime State Model

建议定义统一运行时状态对象，例如：

```text
RunState
- run_id
- session_id
- user_id
- tenant_id
- input_message
- messages
- memory_context
- retrieved_documents
- plan
- tool_calls
- tool_results
- safety_decision
- final_response
- status
- started_at
- finished_at
- usage
- errors
```

建议原则：

- 所有节点都读写同一个共享状态模型
- 用户可见输出和内部推理中间态要分开存储
- 工具结果、检索结果、摘要结果要有明确字段
- 状态对象需要可序列化，方便 checkpoint 和回放

## API Design

### Core Endpoints

- `POST /api/chat`
- `POST /api/chat/stream`
- `GET /api/sessions/{session_id}`
- `GET /api/sessions/{session_id}/messages`
- `POST /api/sessions/{session_id}/messages`
- `GET /api/agents`
- `POST /api/agents/run`
- `GET /api/runs/{run_id}`
- `GET /api/health`

### Example `POST /api/chat`

Request:

```json
{
  "session_id": "sess_123",
  "message": "帮我总结这个项目的架构",
  "agent": "atlas-zero",
  "stream": false,
  "metadata": {
    "user_id": "user_001"
  }
}
```

Response:

```json
{
  "run_id": "run_123",
  "session_id": "sess_123",
  "message": {
    "role": "assistant",
    "content": "..."
  },
  "usage": {
    "input_tokens": 0,
    "output_tokens": 0
  },
  "trace_id": "trace_123"
}
```

### Example `POST /api/chat/stream`

建议返回 `SSE` 事件流，事件类型可包括：

- `run.started`
- `message.delta`
- `tool.started`
- `tool.finished`
- `retrieval.finished`
- `run.completed`
- `run.failed`

这样前端不仅能展示文字流，还能展示 agent 执行过程。

## Suggested Project Structure

```text
app/
  api/
    routes/
      chat.py
      sessions.py
      agents.py
      runs.py
    schemas/
      chat.py
      session.py
      run.py
  core/
    config.py
    logging.py
    security.py
  runtime/
    graph.py
    orchestrator.py
    state.py
    checkpoints.py
  agents/
    stella.py
    shaka.py
    edison.py
    pythagoras.py
    atlas.py
    lilith.py
    york.py
  memory/
    session_store.py
    memory_store.py
    vector_store.py
    summarizer.py
  tools/
    registry.py
    search.py
    browser.py
    knowledge.py
    filesystem.py
  services/
    chat_service.py
    session_service.py
    run_service.py
  observability/
    tracing.py
    metrics.py
    audit.py
  main.py
```

## Persistence Design

### Recommended Storage

- `PostgreSQL`: sessions、messages、runs、audit logs
- `Redis`: cache、rate limit、ephemeral state、queue metadata
- `pgvector` or vector DB: embeddings、document chunks、memory retrieval

### Minimum Tables

- `users`
- `sessions`
- `messages`
- `runs`
- `tool_calls`
- `documents`
- `document_chunks`
- `memories`
- `audit_logs`

## Security and Governance

### Shaka Responsibilities

- 敏感请求识别
- 工具权限控制
- 高风险动作审批
- 输出降级和拒答

### Governance Rules

- 所有工具调用都必须可审计
- 高风险工具必须支持 allowlist 或人工确认
- 模型输出不直接代表工具执行成功
- 用户输入、检索上下文、工具 observation 要分开处理

## Observability

每个 `run` 至少应采集：

- `run_id`
- `trace_id`
- `session_id`
- `agent path`
- `model usage`
- `tool usage`
- `latency`
- `cost`
- `failure reason`

建议从第一版就支持：

- request tracing
- prompt versioning
- run replay
- structured logs

## Delivery Plan

### Phase 1

- `FastAPI` 基础骨架
- `POST /api/chat` 与 `POST /api/chat/stream`
- `Stella -> Shaka -> Edison -> Pythagoras -> Atlas` 最小链路
- session 持久化
- 基础 RAG

### Phase 2

- `Lilith` 扩展探索路径
- `York` 模型路由和预算控制
- 文件上传、文档切分与知识入库
- 更完整的 trace 和 metrics

### Phase 3

- checkpoint 和 resume
- async job queue
- approval workflow
- `MCP` compatibility
- multi-tenant support

## Non-Goals for Early Version

第一阶段不建议过早投入：

- 复杂可视化工作流编辑器
- 过多 agent 角色同时上线
- 过重的插件市场系统
- 过早拆成微服务

建议先把最小闭环跑通，再逐步扩展。
