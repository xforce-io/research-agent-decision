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

## Thesis

Decision agent 作为多智能体员工团队，可与长期助理（如 openclaw）形成互补关系：openclaw 充当经理负责调度，decision agent 承担任务执行。COS-PLAY 展示了决策智能体与技能库智能体解耦并联合训练的可行性，且技能库智能体跨 episode 无状态运行（无持续上下文），为决策智能体独立部署提供初步支持证据。GoA 进一步表明，推理时通过有向图动态路由至专家子智能体（比全量广播节省约 58% 成本）可作为调度链的轻量实现参考，尽管其验证场景限于单轮 QA，尚未覆盖多步任务执行。FLARE 新增关键约束：步进式推理（CoT/ReAct）形式上等价于贪心策略，在长时序任务中系统性失败（首步陷阱率 55.6%，恢复概率 5.4%）；前瞻规划（MCTS + 后向传播）能将规划能力从模型规模中解耦，LLaMA-8B + FLARE 可超越 GPT-4o + 标准推理。OneFlow 提出关键挑战：在同质多智能体工作流（所有 agents 共享同一基础 LLM）中，单智能体执行在精度上匹配甚至超越多智能体，成本降低 5–10×——多智能体结构必要性的前提需要与 Decision Agent 是否属于异质配置（BKN+ContextLoader+Dolphin 专化组件）挂钩。Agent-as-a-Graph 为 BKN 的"逻辑/行动分离"语义解耦提供首份实证支撑：在 MCP 工具/agent 目录上将类型化节点显式分离（区别于 ScaleMCP 的拼接单向量），用类型加权 RRF 检索，Recall@5 提升 14.9%（0.85 vs. 0.74），最优类型权重 αA:αT=1.5:1 跨 8 种 embedding 模型稳健（std≈0.02）；但验证规模仅 527 工具，远低于企业目标的数千级别。MCP-Zero 从相反方向给出第二份证据——让 LLM 主动生成 `<tool_assistant>{server, tool}` 结构化请求触发外部检索：APIBank 多轮 Full 集（48 unique tools）上"模型主笔请求 → top-1 工具"取得 90.32% 精度，token 消耗 −97.52%（6402.2→159.0），相对"用户查询直接检索"基线高 25 个百分点（90.32% vs 65.05%）；与 BKN 思想同向，但 LiveMCPBench 对照显示其被 Agent-as-a-Graph 系统性击败（Recall@5 0.70 vs 0.85），且 GPT-4.1 等强模型上无精度增益（仅省 token），意味着"主动检索"对精度的杠杆主要在弱模型。MAST 在 1642 条开源 MAS 轨迹上以扎根理论编码出 14 类失败模式（FC1 系统设计 44.2% / FC2 智能体间错位 32.3% / FC3 任务校验 23.5%），把论题中"协同失真缺乏学术层面系统性验证"的措辞从"未度量"推进到"已有通用基线、企业场景待对标"。HSC（Heartbeat Scheduling Controller）作为唯一直接以"heartbeat scheduling"为标题的 LLM agent 工作，其形式化框架（macro/micro 双层调度、Dream Mode 三大功能、状态增强 `s_t' = φ(s_t, h_t)`）正面对应 0.8 路线图的 Heartbeat + Cron 设计目标，但实验完全脱离主张（合成数据 + LSTM 监督训练，无 LLM、无 RL、无 baseline），构成**反证案例**——揭示主动调度学术稀缺确实存在，可作为概念语言资源使用，不应被引为机制有效性证据。当前核心问题：Decision Agent 的架构是否构成异质配置（即 OneFlow 等价结论不适用的场景），workers 的内部规划机制如何在企业真实任务中成立，ContextLoader 类型化召回机制能否在更大规模工具集上保持精度，MAST 的 14 个 FM 在 Decision Agent 自家轨迹上的分布是什么，以及 Heartbeat + Reflection 的实证基础需要由 Decision Agent 团队自行建立（学术界目前没有可用证据）。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-05-01 (v8)*
