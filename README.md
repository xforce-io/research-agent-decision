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
| [13](notes/13_toolomni_enabling_open_world_tool_use.md) | ToolOmni: Enabling Open-World Tool Use via Agentic Learning with Proactive Retrieval and Grounded Execution | Tool Management / RL-Trained Proactive Retrieval | high | ✅ deep-read |
| [16](notes/16_g_memory_tracing_hierarchical_memory_for.md) | G-Memory: Tracing Hierarchical Memory for Multi-Agent Systems | Memory / MAS Cross-Trial Self-Evolution | high | ✅ deep-read |
| [17](notes/17_fademem_biologically_inspired_forgetting_for_efficient.md) | FadeMem: Biologically-Inspired Forgetting for Efficient Agent Memory | Memory / Importance-Modulated Decay & Forgetting | high | ✅ deep-read |
| [18](notes/18_governed_memory_a_production_architecture_for.md) | Governed Memory: A Production Architecture for Multi-Agent Workflows | Memory / Org-Level Governed Memory & Schema Lifecycle | high | ✅ deep-read |

## Thesis

Decision Agent 是"治理优先、Harness-first 的企业级决策智能体底座"，5 个 0.8 design goals 须被设计为有治理边界的有限主动模式：goal 2 (Heartbeat+Cron) 与 ISF 存在结构性张力，goal 3/4 须按"内部信息处理 vs 外部副作用"分轨。BKN 语义解耦获 [5][7][11][13] 四条证据线支持，但通用 embedding 在 5k+ 工具规模 NDCG@10 ≤ 42%，工具检索专用训练与 RL 查询改写是工程必需；goal 4 Composer 获 [12] design-time sandbox + ZCL knapsack 原型。Goal 5 Memory 完整工程蓝图由四篇论文堆叠构成：[10] 用户/agent ACL + [16] 跨轨迹 insight 演进 + [17] 重要性调制衰减/冲突仲裁 + [18] 组织级治理路由 + schema lifecycle，但 multi-agent concurrent write conflict 被 [18] 自陈未解决，是 goal 3+5 合并区无任何已读论文系统性回答的工程空白。可证伪：若 BKN + 通用 embedding 在 5k+ 规模 Recall@10 ≥ 50%、[16] 1-hop 最优性在企业异构任务下不成立、或 [17]/[18] LLM 仲裁误分率在合规场景不可接受，则 goal 5 核心假设需修订。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-05-10 (v16)*
