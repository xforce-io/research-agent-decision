# Decision Agent: Multi-Agent Workers Architecture

## Topic

Decision agent 是一种通过 multi-agent workers team 协同完成任务的决策架构，可作为独立组件部署。本课题追踪该架构的技术路径与业界最新进展，重点关注技能管理、协同进化与长期任务执行三个维度。

## Papers

| # | Title | Axis | Priority | Status |
|---|-------|------|----------|--------|
| [01](notes/01_co_evolving_llm_decision_and_skill.md) | Co-Evolving LLM Decision and Skill Bank Agents for Long-Horizon Tasks | Architecture / Co-evolution | high | ✅ deep-read |
| [02](notes/02_graph_of_agents_a_graph_based.md) | Graph-of-Agents: A Graph-based Framework for Multi-Agent LLM Collaboration | Architecture / Graph Routing | med | ✅ deep-read |
| [03](notes/03_why_reasoning_fails_to_plan_a.md) | Why Reasoning Fails to Plan: A Planning-Centric Analysis of Long-Horizon Decision Making in LLM Agents | Architecture / Planning Mechanism | med | ✅ deep-read |

## Thesis

Decision agent 作为多智能体员工团队，可与长期助理（如 openclaw）形成互补关系：openclaw 充当经理负责调度，decision agent 承担任务执行。COS-PLAY 展示了决策智能体与技能库智能体解耦并联合训练的可行性，且技能库智能体跨 episode 无状态运行（无持续上下文），为决策智能体独立部署提供初步支持证据。GoA 进一步表明，推理时通过有向图动态路由至专家子智能体（比全量广播节省约 58% 成本）可作为调度链的轻量实现参考，尽管其验证场景限于单轮 QA，尚未覆盖多步任务执行。FLARE 新增关键约束：步进式推理（CoT/ReAct）形式上等价于贪心策略，在长时序任务中系统性失败（首步陷阱率 55.6%，恢复概率 5.4%）；前瞻规划（MCTS + 后向传播）能将规划能力从模型规模中解耦，LLaMA-8B + FLARE 可超越 GPT-4o + 标准推理。当前核心问题：decision agent workers 的内部规划机制与多智能体协同如何结合，以及该架构能否在开放式真实任务（非游戏、非图遍历）中成立。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-04-27 (v3)*
