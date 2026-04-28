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

## Thesis

Decision agent 作为多智能体员工团队，可与长期助理（如 openclaw）形成互补关系：openclaw 充当经理负责调度，decision agent 承担任务执行。COS-PLAY 展示了决策智能体与技能库智能体解耦并联合训练的可行性，且技能库智能体跨 episode 无状态运行（无持续上下文），为决策智能体独立部署提供初步支持证据。GoA 进一步表明，推理时通过有向图动态路由至专家子智能体（比全量广播节省约 58% 成本）可作为调度链的轻量实现参考，尽管其验证场景限于单轮 QA，尚未覆盖多步任务执行。FLARE 新增关键约束：步进式推理（CoT/ReAct）形式上等价于贪心策略，在长时序任务中系统性失败（首步陷阱率 55.6%，恢复概率 5.4%）；前瞻规划（MCTS + 后向传播）能将规划能力从模型规模中解耦，LLaMA-8B + FLARE 可超越 GPT-4o + 标准推理。OneFlow 提出关键挑战：在同质多智能体工作流（所有 agents 共享同一基础 LLM）中，单智能体执行在精度上匹配甚至超越多智能体，成本降低 5–10×——多智能体结构必要性的前提需要与 Decision Agent 是否属于异质配置（BKN+ContextLoader+Dolphin 专化组件）挂钩。Agent-as-a-Graph 为 BKN 的"逻辑/行动分离"语义解耦提供首份实证支撑：在 MCP 工具/agent 目录上将类型化节点显式分离（区别于 ScaleMCP 的拼接单向量），用类型加权 RRF 检索，Recall@5 提升 14.9%（0.85 vs. 0.74），最优类型权重 αA:αT=1.5:1 跨 8 种 embedding 模型稳健（std≈0.02）；但验证规模仅 527 工具，远低于企业目标的数千级别。当前核心问题：Decision Agent 的架构是否构成异质配置（即 OneFlow 等价结论不适用的场景），workers 的内部规划机制如何在企业真实任务中成立，以及 ContextLoader 类型化召回机制能否在更大规模工具集上保持精度。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-04-29 (v5)*
