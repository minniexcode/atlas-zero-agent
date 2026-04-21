# Atlas Zero Agent

`Atlas Zero Agent` 是一个通用的 chatbox agent 项目，目标是结合 `LangChain` 的组件化能力与更偏 agent runtime 的任务执行特性，构建一个可扩展、可编排、可观测的智能体系统。

## Goals

- 通用聊天入口
- 多 agent 协同
- 工具调用与任务执行
- Memory 与 RAG
- FastAPI 服务化
- 可观测与可治理

## Agent Naming

项目使用《海贼王》蛋头岛 `Vegapunk satellites` 的命名体系：

- `Stella`: 主控入口
- `Atlas`: 执行型 agent
- `Edison`: 规划与编排 agent
- `Pythagoras`: 记忆与知识 agent
- `Shaka`: 安全与规则 agent
- `Lilith`: 探索与实验 agent
- `York`: 资源与成本 agent

## Docs

- 设计草案：`atlas-zero-agent-design.md`

## Planned Stack

- `FastAPI`
- `LangChain`
- `LangGraph`
- `PostgreSQL`
- `Redis`
- `pgvector`

## Status

当前仓库处于早期设计阶段，优先完善架构、agent 分工与最小可运行骨架。
