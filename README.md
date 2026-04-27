# Decision Agent: Multi-Agent Workers Architecture

## Topic

Decision agent 是一种通过 multi-agent workers team 协同完成任务的决策架构，可作为独立组件部署。本课题追踪该架构的技术路径与业界最新进展，重点关注技能管理、协同进化与长期任务执行三个维度。

## Papers

| # | Title | Axis | Priority | Status |
|---|-------|------|----------|--------|
| [01](notes/01_co_evolving_llm_decision_and_skill.md) | Co-Evolving LLM Decision and Skill Bank Agents for Long-Horizon Tasks | Architecture / Co-evolution | high | ✅ deep-read |
| [02](notes/02_graph_of_agents_a_graph_based.md) | Graph-of-Agents: A Graph-based Framework for Multi-Agent LLM Collaboration | Architecture / Graph Routing | med | ✅ deep-read |

## Thesis

Decision agent 作为多智能体员工团队，可与长期助理（如 openclaw）形成互补关系：openclaw 充当经理负责调度，decision agent 承担任务执行。COS-PLAY 展示了决策智能体与技能库智能体解耦并联合训练的可行性，且技能库智能体跨 episode 无状态运行（无持续上下文），为决策智能体独立部署提供初步支持证据。GoA 进一步表明，推理时通过有向图动态路由至专家子智能体（比全量广播节省约 58% 成本）可作为调度链的轻量实现参考，尽管其验证场景限于单轮 QA，尚未覆盖多步任务执行。当前关注点是该架构的技术路径是否能在非游戏的真实任务环境中成立。

See [`.researcher/thesis.md`](.researcher/thesis.md) for the full working thesis.

## Depth

- [`notes/00_research_landscape.md`](notes/00_research_landscape.md) — living synthesis of all read papers
- [`.researcher/thesis.md`](.researcher/thesis.md) — working thesis and research posture

*Last Updated: 2026-04-27*
