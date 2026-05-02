# Decision Agent: Multi-Agent Workers Architecture

## Topic

Decision agent 是一种通过 multi-agent workers team 协同完成任务的决策架构，可作为独立组件部署。本课题追踪该架构的技术路径与业界最新进展，重点关注技能管理、协同进化与长期任务执行三个维度。

## Papers

| # | Title | Axis | Priority | Status |
|---|-------|------|----------|--------|
| [01](notes/01_co_evolving_llm_decision_and_skill.md) | Co-Evolving LLM Decision and Skill Bank Agents for Long-Horizon Tasks | Architecture / Co-evolution | high | ✅ deep-read |
| [02](notes/02_graph_of_agents_a_graph_based.md) | Graph-of-Agents: A Graph-based Framework for Multi-Agent LLM Collaboration | Architecture / Graph Routing | med | ✅ deep-read |
| [03](notes/03_why_reasoning_fails_to_plan_a.md) | Why Reasoning Fails to Plan: A Planning-Centric Analysis of Long-Horizon Decision Making in LLM Agents | Architecture / Planning Mechanism | med | ✅ deep-read |
| [04](notes/04_rethinking_the_value_of_multi_agent.md) | Rethinking the Value of Multi-Agent Workflow: A Strong Single Agent Baseline | Architecture / Single-Agent Baseline | high | ✅ deep-read |
| [05](notes/05_agent_as_a_graph_knowledge_graph.md) | Agent-as-a-Graph: Knowledge Graph-Based Tool and Agent Retrieval for LLM Multi-Agent Systems | Tool Management / Catalog Retrieval | high | ✅ deep-read |
| [06](notes/06_why_do_multi_agent_llm_systems.md) | Why Do Multi-Agent LLM Systems Fail? | Architecture / Failure Diagnostics | high | ✅ deep-read |
| [07](notes/07_mcp_zero_active_tool_discovery_for.md) | MCP-Zero: Active Tool Discovery for Autonomous LLM Agents | Tool Management / Active Discovery | high | ✅ deep-read |
| [08](notes/08_simulating_human_cognition_heartbeat_driven_autonomous.md) | Simulating Human Cognition: Heartbeat-Driven Autonomous Thinking Activity Scheduling for LLM-based AI systems | Runtime / Heartbeat Scheduling | high | ✅ deep-read |
| [09](notes/09_llm_based_multi_agent_blackboard_system.md) | LLM-based Multi-Agent Blackboard System for Information Discovery in Data Science | Architecture / Broadcast-and-Self-Select | high | ✅ deep-read |
| [10](notes/10_collaborative_memory_multi_user_memory_sharing.md) | Collaborative Memory: Multi-User Memory Sharing in LLM Agents with Dynamic Access Control | Memory / Cross-Session Layered Access Control | high | ✅ deep-read |
| [11](notes/11_retrieval_models_aren_t_tool_savvy.md) | Retrieval Models Aren't Tool-Savvy: Benchmarking Tool Retrieval for Large Language Models | Tool Management / Retrieval Benchmarking | high | ✅ deep-read |
| [12](notes/12_automated_composition_of_agents_a_knapsack.md) | Automated Composition of Agents: A Knapsack Approach for Agentic Component Selection | Composer / Sandbox-Validated Knapsack Selection | high | ✅ deep-read |

## Thesis

- **核心定位**：Decision Agent 是"治理优先、Harness-first 的企业级决策智能体底座"——通过 BKN + ISF + TraceAI + ContextLoader 四支柱垂直整合，价值锚点是 Dify/Coze 无法企及的语义治理深度，而非通用编排市场。
- **0.8 增强主张与张力**：5 个 design goals（模板 / Heartbeat+Cron / Shared Workspace / Composer / 层级化 Memory）需被设计为"有治理边界的有限主动模式"——goal 2 主动 Runtime 与 ISF 治理存在结构性张力；goal 3/4 须按"内部信息处理 vs 外部副作用"分轨。
- **当前证据图**：BKN 语义解耦获 [5][7] 双证支持（+14.9% Recall@5 / +25 pp），但 [11] 进一步揭示通用 IR 在工具检索上 Completeness@10 ≤ 50%、必须做工具检索专用微调；MAST [6] 把"协同失真"推进到"已有通用基线"；OneFlow [4] 折叠等价边界已被 [9][12] 双反证细化（上下文 fit + supervisor delegation 容量）；[12] 为 goal 4 Composer 提供 design-time sandbox + ZCL knapsack pipeline 原型。
- **可证伪点**：若 Decision Agent 经审计属同质配置、5 个增强能力可由通用框架替代、自家 FC2 ≪ 32.3%、或 BKN + 通用 embedding 在 5k+ 规模 Recall@10 ≥ 50%，则"Harness-first 垂直整合"主张需重估。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-05-02 (v12)*
