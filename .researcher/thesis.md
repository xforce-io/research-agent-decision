# Thesis

> Working thesis lives here. Researcher reads this every run and uses it to
> decide whether new material supports / extends / challenges / is orthogonal
> to your current view. Researcher will *report* contradictions but never
> edit this file — thesis changes are always your decision.

## Working thesis

Decision Agent 的核心架构挑战不是"让单个 worker 更聪明"，而是在保证可观测性、可审计性的前提下，使多智能体 workers team 能够可靠完成端到端的企业业务任务。当前技术栈（BKN 语义底座 + ContextLoader 动态加载 + Dolphin 编排引擎）解决了上下文管理和语义对齐问题。其中 workers 协同失真已有 MAST (Cemri et al. 2025, arxiv:2503.13657) 这类系统性失败模式分类——FC2 Inter-Agent Misalignment 占 32.3%，FM-1.3 Step Repetition 15.7%，FM-1.5 Unaware of Termination 12.4%，并提供 agentdash 可复用诊断工具。Decision Agent 的差异化命题因此从"做学术验证"重定位为：(a) 企业长链任务下 FC1/FC2/FC3 权重是否与开源 MAS 一致；(b) Dolphin 状态机对 FM-1.3/FM-1.5 等高频失败模式的工程级缓解能否被量化。长链规划退化和多智能体调度仲裁仍缺乏企业场景的系统性验证。该论点可被证伪——若发现主流解决方案不需要专门的多智能体协调机制，靠单一足够强的 LLM 端到端完成即可，则需修正架构优先级。

多智能体 workers 的协同质量受规划机制的结构性约束。步进式推理（ReAct/CoT）在形式上等价于贪心策略，在需要超过若干步序列决策的任务上将系统性失败——这与 Decision Agent 的长链业务任务（"查询-分析-撰写-校验-审批-落地"等多步 AutoFlow）直接相关。前瞻性规划（MCTS 类方法）从理论上可修复规划深度问题，但会引入延迟代价。Decision Agent 需要在规划质量与响应延迟之间做明确工程取舍，目前该取舍点尚无经过企业场景验证的基准。

ContextLoader 的动态按需加载机制（工具定义上下文占用降低 90%+）是 Decision Agent 区别于通用 Agent 的核心工程差异。工具幻觉是一个反直觉难题：模型推理能力越强，在工具调用上越容易自信地犯错。BKN 语义解耦（逻辑/行动分离 + 业务对象关联）是当前主要防御机制，但其在超大工具集（企业数千工具）场景下的有效性尚缺乏系统性验证，需要关注语义精确召回与覆盖率的工程平衡点。

Openclaw 与 Decision Agent 的关系是"经理-员工"调度结构：openclaw 提供持续上下文与任务调度入口，Decision Agent 作为独立组件执行具体任务。关键可证伪点：Decision Agent workers 是否需要持续依附 openclaw 上下文才能在长链任务中闭环——若是，则二者耦合程度远超"调度入口"，更接近"持续上下文依赖"，分层边界需要重新评估。

## Design Context

**正在构建的系统**：KWeaver Decision Agent 平台，核心技术栈为 BKN + ContextLoader + Dolphin 编排引擎 + ISF 安全层，目标是让企业多智能体 workers team 可靠完成端到端业务决策任务。详见 `references/01_decision_agent_features.md` 和 `references/02_decision_agent_tool_management.md`。

**当前最大的研究缺口**：
1. Workers team 多智能体协调机制——Dolphin 状态机如何在长链任务中防止协作失真，缺乏学术对标
2. 长链规划退化——Plan-Reason-Act 在 >5 步任务中的规划质量无学术基准
3. 工具幻觉的语义解耦防御——BKN 机制的有效性边界尚未系统验证

**成功标准**：为 Decision Agent 技术路线图提供可操作的学术支撑，识别 Dolphin 调度设计与业界最优实践的具体差距，并为 ContextLoader 工具召回机制提供可参考的精确度-覆盖率基准。

## Taste

- **偏好有明确 Plan-Reason-Act 分层和可观测性的研究**，而非端到端黑盒方法——Decision Agent 全链路可审计要求每层输出必须独立可追溯
- **偏好在真实多步工具调用任务中验证的工作**，而非单轮 QA 或玩具环境——Decision Agent 的核心价值在多步 AutoFlow 闭环，单轮推理场景无法迁移
- **偏好有工具检索/召回机制分析的研究**——ContextLoader 的核心挑战是大规模工具集的精准按需加载，工具选择机制是直接相关问题
- **偏好有明确 worker 独立性边界分析的研究**——RQ3 的核心：workers 在无持续外部上下文注入条件下能否独立闭环
- **偏好有协同失真量化的多智能体工作**——信息在 Agent 间传递中被压缩、丢关键约束是 Decision Agent 的已知工程痛点，需要机制而非定性描述

## Anti-patterns

- **单一 LLM 能力提升论文（更大模型、更好 prompt）**——Decision Agent 的瓶颈是协同架构和上下文管理，不是单 agent 推理能力
- **仅在游戏/形式化状态空间环境验证的规划方法**——Decision Agent 工作在非确定性企业数据场景，确定性状态转移假设不适用
- **不涉及上下文管理的多智能体协同研究**——上下文窗口是 Decision Agent 最核心的工程约束，忽略它的协同方案无法直接采用
- **纯基准测评没有机制分析**——KWeaver 需要可移植的机制（召回策略、调度算法），不是不可迁移的 benchmark 数字
- **不区分 worker 独立性和 manager 依赖性的研究**——模糊这个区分的论文对 openclaw-Decision Agent 分层架构决策无贡献
- **"靠 prompt 解决工具幻觉"的工作**——BKN 语义结构化解耦才是 Decision Agent 采用的路径，纯 prompt 方案不可规模化

## Examples

好的收录（参照）：
- `notes/01_co_evolving_llm_decision_and_skill.md` — 技能库协同进化机制直接对应 ContextLoader 工具管理问题
- `notes/03_why_reasoning_fails_to_plan_a.md` — 步进式推理结构性局限的严格形式化，直接挑战 Dolphin 状态机规划假设

待收录优先级高的类型：
- 工具检索精确召回（向量 vs 语义结构化对比）
- 多智能体协调中信息保真度量化
- LLM agent 在企业真实任务中的长链规划失败分析
