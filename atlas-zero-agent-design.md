# Atlas Zero Agent 设计草案

## 1. 项目目标

`atlas-zero-agent` 目标是做成一个通用的 `chatbox agent` 平台：

- 对用户来说，它首先是一个好用的聊天入口，支持普通对话、工具调用、长期上下文和知识检索。
- 对系统来说，它不只是一个单轮聊天接口，而是一个可编排、可扩展、可观测的 agent runtime。
- 对工程实现来说，它希望同时吸收 `LangChain` 的组件化能力，以及 `harness agent` 一类系统在任务执行、工具编排、状态管理上的特性。

一句话定位：

> 一个以聊天为入口、以 agent runtime 为核心、以多人格分工为产品语言的通用智能体系统。

## 2. 贝加庞克分身设定

《海贼王》蛋头岛篇里，贝加庞克把自己的不同人格侧面拆成了多个 `satellites`。本体叫 `Stella`，分身共有 6 个：

- `Shaka`
- `Lilith`
- `Edison`
- `Pythagoras`
- `Atlas`
- `York`

这些分身本质上不是完全独立的新角色，而是同一个超级科学家不同侧面的拆分与外化。这一点和 agent 设计非常契合，因为一个完整 agent 往往也会被拆成：规划、执行、记忆、安全、探索、资源调度等不同能力模块。

## 3. 分身名称、性格与来源

下面的“来源”分两层理解：

- 第一层是 `One Piece` 设定中的人格侧面。
- 第二层是角色名字在现实世界中可能对应的神话、历史人物、哲学家或文化意象。

需要说明的是：作品中“人格职责”是明确设定；而“名字来源”很多属于高概率命名致敬或常见解读，不一定都有作者正式逐条说明。

### 3.1 Shaka

- 作品内定位：`Good`，代表“善”
- 角色气质：冷静、克制、规则感强、像总控和秩序维护者
- 常见名字来源解读：
  - 最常见的理解是来自 `Shaka`，也就是日语中对释迦牟尼的称呼之一，关联佛教中的理性、觉悟、秩序与道德判断。
  - 也有一些二级解读会联系到其他历史人物或天体命名，但核心印象仍然是“理性、正面、克制”。
- 适合映射的 agent 角色：
  - 安全策略 agent
  - 系统规则 agent
  - 审核与 guardrails agent
  - 风险判定 agent

为什么适合：

- 一个通用 chat agent 不能只有能力，还必须有边界。
- `Shaka` 很适合做“是否允许继续执行”“是否触发敏感操作确认”“是否需要降级回答”的决策层。

### 3.2 Lilith

- 作品内定位：`Evil`，代表“恶”
- 角色气质：激进、冒险、追求结果、手段不那么保守
- 常见名字来源解读：
  - `Lilith` 在美索不达米亚与犹太传统里常被视为带有黑暗、反叛、原初女性恶魔等色彩的形象。
  - 在现代流行文化中，`Lilith` 也常被用来代表“不服从、危险、诱惑、边界突破”。
- 适合映射的 agent 角色：
  - 探索型 agent
  - 实验型 agent
  - 自主搜索与假设验证 agent
  - “尝试不同路径” 的发散 agent

为什么适合：

- 很多 agent 系统失败，不是因为不会回答，而是不会“试”。
- `Lilith` 可以承担更激进的探索任务，例如多路搜索、失败重试、方案比较、信息侦察。
- 但她不应拥有最终批准权，最好受 `Shaka` 约束。

### 3.3 Edison

- 作品内定位：`Think`，代表“想”
- 角色气质：灵感驱动、发明导向、快速出点子
- 常见名字来源解读：
  - 明确对应现实中的发明家 `Thomas Edison`。
  - 象征发明、工程实现、原型驱动、把抽象想法落成机制。
- 适合映射的 agent 角色：
  - 规划型 agent
  - 工具编排 agent
  - 工作流生成 agent
  - 任务分解与步骤生成 agent

为什么适合：

- 你的系统如果想兼具 `LangChain` 与 harness 风格，就需要一个专门负责“把用户目标转为可执行步骤”的角色。
- `Edison` 很适合做 planner，决定先检索、再调用工具、再总结，或者什么时候需要拆任务并回写状态。

### 3.4 Pythagoras

- 作品内定位：`Wisdom`，代表“知”
- 角色气质：整理、分析、汇总、把知识结构化
- 常见名字来源解读：
  - 明确对应古希腊哲学家、数学家 `Pythagoras`。
  - 象征数学、结构、抽象、归纳、规律与知识体系。
- 适合映射的 agent 角色：
  - 知识库 agent
  - `RAG` agent
  - memory agent
  - 数据分析与事实整理 agent

为什么适合：

- 一个“通用 chatbox agent”如果没有稳定的记忆和知识管理，就很容易变成只会临场生成文本的壳。
- `Pythagoras` 非常适合负责检索、重排、引用、会话摘要、长期记忆写入与知识组织。

### 3.5 Atlas

- 作品内定位：`Violence`，代表“暴”
- 角色气质：行动直接、执行果断、解决问题优先
- 常见名字来源解读：
  - 最直观来源是希腊神话中的巨人 `Atlas`，常见印象是“扛起天空的人”。
  - 这个名字还有“地图册、承载世界结构”的延伸含义，但在角色气质上更突出的是力量、承担、推进。
- 适合映射的 agent 角色：
  - 执行型 agent
  - 行动派 agent
  - 工具调用 agent
  - 任务落地 agent

为什么适合：

- 如果 `Edison` 负责想，`Atlas` 就负责做。
- 她最适合做实际执行层，例如调用工具、跑工作流、操作外部服务、提交任务结果。
- 对你的项目名来说，`Atlas` 也很强，因为它兼具力量感和工程执行感。

### 3.6 York

- 作品内定位：`Greed`，代表“欲”
- 角色气质：偏向资源占用、需求满足、为整体系统承担身体性与消耗性工作
- 常见名字来源解读：
  - `York` 的来源没有前几个那么明确，常见解读并不统一。
  - 从作品功能上看，她更像“欲望 / 需求 / 资源消耗”的人格化承载体。
  - 也有一些解读会把它和天体、地名或科学人物关联，但都不如她在故事中的“欲望代理”功能重要。
- 适合映射的 agent 角色：
  - 资源调度 agent
  - 队列与吞吐 agent
  - 成本控制 agent
  - 配额、缓存、优先级管理 agent

为什么适合：

- 一个能跑起来的 agent 系统，除了会想、会做，还要知道“资源够不够、应该先跑谁、哪个模型值得调用、要不要缓存”。
- `York` 非常适合负责 token 成本、模型路由、任务队列、优先级、配额和后台任务管理。

### 3.7 Stella

- 作品内定位：本体
- 名称含义：`Stella` 在拉丁语中有“星”之意
- 适合映射的系统角色：
  - 主控人格
  - 对外统一身份
  - 路由入口
  - 多 agent 聚合层

为什么适合：

- 用户不一定需要直接知道背后有多少子 agent。
- 对外可以统一叫 `Atlas Zero Agent`，内部再由 `Stella` 作为总入口路由到不同分身能力。

## 4. 推荐的命名分工方案

如果你准备把这套世界观真正用进产品，我建议不要把这些名字只当作皮肤，而是让它们对应真实职责。

### 4.1 推荐的一号方案

- `Stella`：主控入口 / orchestrator
- `Atlas`：执行型 agent
- `Edison`：规划与工具编排 agent
- `Pythagoras`：记忆、RAG、知识整理 agent
- `Shaka`：安全、审核、边界控制 agent
- `Lilith`：探索、实验、侦察 agent
- `York`：资源、成本、队列调度 agent

这个方案的优点：

- 人设和能力匹配度高。
- 后续做多 agent 扩展时可自然拆模块。
- 即使后面只实现其中 2 到 3 个角色，也不会浪费命名体系。

### 4.2 适合你的产品表达方式

对外产品文案可以写成：

- `Atlas Zero` 是统一对话入口。
- 内部由多个 `satellite agents` 协同工作。
- 用户默认只看到一个 chatbox，但系统会自动路由到最合适的能力模块。

这样既保留世界观，也不会把产品做得过度复杂。

## 5. 项目架构建议

你的目标不是单纯搭一个聊天接口，而是做一个“通用 agent 平台”。所以架构上建议分成六层。

### 5.1 总体分层

1. `Interface Layer`
2. `Application Layer`
3. `Agent Runtime Layer`
4. `Knowledge and Memory Layer`
5. `Tool and Integration Layer`
6. `Observability and Governance Layer`

### 5.2 各层职责

#### Interface Layer

建议技术：`FastAPI`

职责：

- 提供聊天接口
- 提供流式响应接口，建议支持 `SSE`
- 提供会话管理接口
- 提供 agent 配置与调试接口
- 未来可扩展 `WebSocket`

建议接口：

- `POST /api/chat`
- `POST /api/chat/stream`
- `GET /api/sessions/{session_id}`
- `POST /api/sessions/{session_id}/messages`
- `GET /api/agents`
- `POST /api/agents/run`
- `GET /api/health`

#### Application Layer

职责：

- 处理 API 请求与响应模型
- 维护会话上下文
- 调用 orchestrator
- 统一错误处理
- 做鉴权、租户、多用户隔离

这一层尽量不要直接写复杂推理逻辑，而是作为业务编排层。

#### Agent Runtime Layer

建议技术：`LangGraph` + `LangChain`

职责：

- 管理多 agent 状态流转
- 管理 planner、executor、memory、safety 等节点
- 支持可中断、可恢复、可追踪执行
- 统一处理工具调用结果
- 支持多步任务执行

为什么这里建议 `LangGraph` 而不是只用 `LangChain`：

- `LangChain` 适合组件和工具接线。
- `LangGraph` 更适合处理 agent 的状态机、分支、恢复、checkpoint 和长任务。
- 你的目标已经超出简单 chain，更接近一个小型 runtime。

#### Knowledge and Memory Layer

职责：

- 会话短期记忆
- 用户长期记忆
- 文档知识库检索
- 摘要压缩
- 事实缓存和向量检索

建议拆成三类存储：

- `Session Store`：保存对话消息和运行状态
- `Memory Store`：保存用户长期偏好、画像、任务历史
- `Vector Store`：保存 RAG 文档嵌入和检索索引

适合由 `Pythagoras` 主导。

#### Tool and Integration Layer

职责：

- 封装本地工具
- 封装外部 API
- 封装搜索、数据库、文件、浏览器等工具
- 统一工具协议和权限边界

这层适合由 `Atlas` 执行、由 `Edison` 编排。

如果后面要做得更像现代 agent 平台，可以预留：

- `MCP` 兼容层
- sandbox 执行层
- 第三方 connectors

#### Observability and Governance Layer

职责：

- trace
- prompt 版本管理
- tool call 日志
- token 和成本统计
- 审计与安全策略
- 人工接管与审批

这层很关键，因为真正可用的 agent 系统一定需要可观测性，而不是只有“能回话”。

`Shaka` 和 `York` 最适合共同负责这一层。

## 6. 推荐的内部多 agent 结构

如果以“单 chatbox，多内部人格”的方式实现，建议这样组织：

### 6.1 最小闭环版本

- `Stella Router`
- `Edison Planner`
- `Atlas Executor`
- `Pythagoras Memory`
- `Shaka Guard`

这是最适合第一阶段落地的组合。

对应流程：

1. 用户输入进入 `Stella Router`
2. `Shaka Guard` 先做风险检查与权限判断
3. `Edison Planner` 生成执行步骤
4. `Pythagoras Memory` 补充上下文、检索记忆与知识
5. `Atlas Executor` 调用工具或执行动作
6. `Stella Router` 汇总结果并输出给用户

### 6.2 第二阶段扩展

新增：

- `Lilith Explorer`
- `York Resource Manager`

用途：

- `Lilith` 用于多路探索、网络搜索、候选方案试探、故障 fallback
- `York` 用于模型路由、预算控制、优先级与后台任务调度

## 7. 推荐的目录结构

如果你准备用 `FastAPI + LangGraph + LangChain`，我建议从下面这个目录开始，而不是一上来就堆太多文件。

```text
atlas-zero-agent/
  app/
    api/
      routes/
        chat.py
        sessions.py
        agents.py
      schemas/
        chat.py
        session.py
        agent.py
    core/
      config.py
      logging.py
      security.py
    runtime/
      orchestrator.py
      state.py
      graph.py
    agents/
      stella.py
      atlas.py
      edison.py
      pythagoras.py
      shaka.py
      lilith.py
      york.py
    memory/
      session_store.py
      memory_store.py
      vector_store.py
      summarizer.py
    tools/
      registry.py
      web_search.py
      browser.py
      filesystem.py
      knowledge.py
    services/
      chat_service.py
      session_service.py
      agent_service.py
    observability/
      tracing.py
      metrics.py
      audit.py
    main.py
  docs/
    architecture.md
    agents.md
  tests/
    test_chat_api.py
    test_runtime_flow.py
    test_memory.py
  pyproject.toml
  README.md
```

这个结构的重点：

- `api` 只管接口
- `runtime` 只管状态流转和 orchestrator
- `agents` 只放角色级逻辑
- `memory` 和 `tools` 独立，方便后续扩展
- `observability` 独立，避免后期难补

## 8. 推荐的技术栈

### 8.1 后端核心

- `FastAPI`：对外 API
- `Pydantic`：请求响应模型与配置
- `LangChain`：模型、prompt、tools、retriever 抽象
- `LangGraph`：多 agent 编排与状态管理
- `Uvicorn`：服务运行

### 8.2 存储层

- `PostgreSQL`：会话、用户、任务、审计
- `Redis`：缓存、队列、短期状态
- `pgvector` 或独立向量库：RAG 检索

### 8.3 观测层

- `LangSmith`：链路追踪和调试
- `OpenTelemetry`：统一 trace/metrics
- `Prometheus` + `Grafana`：运行指标

### 8.4 前端

- 如果先做后端验证：先不急着做复杂前端
- 如果要快速出 chatbox：
  - `Next.js`
  - 或直接参考 `langchain-ai/agent-chat-ui`

## 9. 产品能力建议

你说的是“通用 chatbox agent”，那我建议第一版不要贪多，优先做下面这些能力：

### 第一阶段必须有

- 多轮对话
- 流式输出
- session 持久化
- 工具调用
- 基础 RAG
- 安全拦截
- trace 日志

### 第二阶段再加

- 多 agent 协同
- 用户长期记忆
- 文件上传解析
- 后台异步任务
- 模型路由
- 成本统计与预算限制

### 第三阶段再加

- 人工审批
- MCP 工具接入
- 插件市场
- 多租户
- 可视化工作流

## 10. 命名建议结论

如果你希望这个项目既有故事感，又能长期演化，我建议这样定：

- 项目总名：`atlas-zero-agent`
- 对外产品名：`Atlas Zero`
- 内部 orchestrator：`Stella`
- 核心执行 agent：`Atlas`
- 规划 agent：`Edison`
- 记忆与知识 agent：`Pythagoras`
- 安全与审查 agent：`Shaka`
- 探索与实验 agent：`Lilith`
- 资源与预算 agent：`York`

这样命名的好处是：

- 有统一世界观
- 每个名字都有明确职责
- 后续做文档、UI、日志、trace 时都很好表达
- 未来如果你要把系统从单 agent 进化到多 agent，也不需要重命名

## 11. 最推荐的启动路线

如果现在就开始做，我建议按这个顺序推进：

1. 先做 `FastAPI` 聊天接口和 `SSE` 流式输出
2. 接入一个基础 `LangChain` chat model
3. 用 `LangGraph` 搭建 `Stella -> Shaka -> Edison -> Pythagoras -> Atlas` 的最小图
4. 接入 session store 和基础 memory
5. 接入 2 到 3 个最常用工具
6. 增加 trace、审计和成本统计
7. 最后再扩展 `Lilith` 和 `York`

## 12. 一句话设计原则

`Atlas Zero` 不应该只是“会聊天的模型壳”，而应该是：

> 以 `Atlas` 为执行核心、以 `Stella` 为统一入口、以多分身协作为内部机制的通用 agent runtime。
