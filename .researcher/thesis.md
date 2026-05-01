# Thesis

> Working thesis lives here. Researcher reads this every run and uses it to
> decide whether new material supports / extends / challenges / is orthogonal
> to your current view. Researcher will *report* contradictions but never
> edit this file — thesis changes are always your decision.

## Working thesis

Decision Agent 的核心定位是**治理优先、Harness-first 的企业级决策智能体底座**——通过 BKN 语义网络（业务护栏）、ISF（安全边界）、TraceAI（审计底座）、ContextLoader（高质量上下文）四个支柱的垂直整合，在企业核心业务场景实现可观测、可审计、可评测的多步决策闭环。这不是又一个 Agent 框架（不与 LangGraph/CrewAI 竞争通用编排市场），而是面向强监管行业的"专有 Agent 底座"——其价值锚点是 Dify/Coze 等浅层编排无法企及的语义治理深度，以及 Palantir AIP 之外更低部署门槛的同等治理能力。

经过 0.7.x 版本迭代，Decision Agent 已具备 BKN 语义建模、ContextLoader 精准召回、Execution Factory 沙盒执行、TraceAI 审计留痕的基础能力，并在 0.8 路线图中明确 4 个结构性短板（非 7×24 设计 / 使用门槛高 / 协同信息孤岛 / 记忆体系不完整）。下一阶段需通过 5 个增强能力（开箱即用模板 / Heartbeat+Cron 常驻 Runtime / Shared Workspace / Composer 轻量多智能体组合 / 跨会话层级化 Memory）从"被动响应模式"演进到"企业生产级多智能体系统"。该论点可被证伪——若发现强监管企业场景下"治理深度"并非主要付费动力，或若 5 个增强能力中任一被证明可由通用框架 + 简单封装替代，则需重新评估"Harness-first 垂直整合"的战略主张。

**协同失真已有 MAST (arxiv:2503.13657) 系统性失败模式分类**——FC2 Inter-Agent Misalignment 占 32.3%，FM-1.3 Step Repetition 15.7%，FM-1.5 Unaware of Termination 12.4%。Decision Agent 的差异化命题因此从"做学术验证"重定位为：(a) 企业长链任务下 FC1/FC2/FC3 权重是否与开源 MAS 一致；(b) Shared Workspace + BKN 共享 Memory 对 FC2 的工程级缓解能否被量化；(c) Dolphin 状态机对 FM-1.3/FM-1.5 等高频失败模式的工程级缓解能否被量化。

**多智能体 workers 的协同质量受规划机制的结构性约束**。步进式推理（ReAct/CoT）在形式上等价于贪心策略（FLARE, arxiv 已读 [3]），在需要超过若干步序列决策的任务上将系统性失败——这与 Decision Agent 的长链业务任务（"查询-分析-撰写-校验-审批-落地"等多步 AutoFlow）直接相关。前瞻性规划（MCTS 类方法）从理论上可修复规划深度问题，但会引入延迟代价。Decision Agent 需要在规划质量与响应延迟之间做明确工程取舍，目前该取舍点尚无经过企业场景验证的基准。

**ContextLoader 的动态按需加载机制（工具定义上下文占用降低 90%+）是 Decision Agent 区别于通用 Agent 的核心工程差异**。工具幻觉是一个反直觉难题：模型推理能力越强，在工具调用上越容易自信地犯错——但需注意 MCP-Zero (arxiv:2506.01056 [7]) 的边界条件：在 GPT-4.1 等强模型 + needle-in-haystack（≤2,797 tools）场景下 baseline 已饱和，"主动检索"的精度增益主要落在弱模型上，强模型档位的实际增益更可能落在 token 成本侧而非精度侧。BKN 语义解耦（逻辑/行动分离 + 业务对象关联）是当前主要防御机制，已有两条独立证据线支持（[5] Agent-as-a-Graph 类型化召回 +14.9% Recall@5；[7] MCP-Zero 模型主笔结构化请求 +25 个百分点），但验证规模均远低于企业 5k–10k 工具目标，尚需自家 BKN 工具集复测。

**Decision Agent 与 OpenClaw 的关系不是"经理-员工"层级，而是"治理深度差异 + 双方都具备 7×24 能力"的并列形态**（参见 references/03 §一）。Decision Agent 是治理优先的编排底座（BKN+ISF+TraceAI 完整治理链路），OpenClaw 是轻量级个人助理（强调快速响应与个人效率）。Decision Agent 也可独立使用，不依赖 OpenClaw 即可完成单体复杂任务——这一独立闭环已被产品定位明确，剩余研究问题是工程层面的 worker 独立性边界（哪些规划决策依赖 manager 状态、哪些可由 BKN + 当前 trace 计算）。

**5 个 design goals 不是默认相互正交，goal 2 (Heartbeat + Cron 主动 Runtime) 与 thesis 治理优先核心定位存在结构性张力**（由 [8] HSC 阅读触发的反读，notes/08）。Always-on 主动调度的"空闲态自主认知活动"（如 HSC 的 Dream Mode、Reflection 自主目标设定）在结构上违反 ISF/TraceAI 的"action provenance must be explicit"治理约束——主动 agent 越主动，治理越被动。因此 goal 2 不是单纯的"补足 7×24 能力"，必须被设计为**有治理边界的主动模式（governance-aware bounded autonomy）**：(a) 主动行为必须有显式调度策略来源（cron 表达式 / 用户授权的 Reflection 触发器 / Composer Plan 派生），不允许 intrinsic goal generation 进入生产路径；(b) 主动行为必须 TraceAI-first（先写审计再执行）；(c) 涉及外部副作用的主动动作必须经过 ISF + Human-in-the-loop。Reflection 类能力可保留但限定在"建议生成"层而非"自主执行"层。**学术稀缺信号需要重新解读**：原假设"稀缺=机会窗口"被 [8] 部分证伪——更可能的解释是"该问题在企业场景下可由 cron + ReAct + 治理围栏简单组合解决"（同 OneFlow 对多 agent 必要性的证伪逻辑）；goal 2 的工程优先级因此应从"补足空白"调整为"在治理边界内实现有限主动模式"。

## Design Context

**正在构建的系统**：KWeaver Decision Agent 平台，核心技术栈为 BKN + ContextLoader + Dolphin 编排引擎 + ISF 安全层 + TraceAI 审计 + Execution Factory 沙盒执行。详见：
- `references/01_decision_agent_features.md` — 产品特性、五大技术目标、五大业务价值
- `references/02_decision_agent_tool_management.md` — 工具管理与 BKN 语义解耦方案
- `references/03_decision_agent_roadmap.md` — 0.8 路线图：4 短板 + 5 增强需求 + 4 竞品差异化 + 5 非目标

**0.8 路线图明确的 5 个增强能力（design goals）**：

1. **开箱即用（§3.1）**：预置运行模式（ReAct / Plan-Execute）+ 场景模板库 + 渐进式学习路径，让用户不写 DPH 脚本也能跑通业务场景
2. **Heartbeat + Cron（§3.2）**：轻量常驻 Runtime + 定时调度引擎 + 任务状态持久化 + Reflection 能力，让 Agent 从被动响应演进为主动 7×24 数字员工
3. **Shared Workspace（§3.3）**：Session/Task 级共享工作区 + 引用传递机制 + ISF 权限控制，根本性降低多 Agent 协作的上下文压力
4. **Composer（§3.4）**：基于已有 Agent/Template/Skill 的轻量多智能体组合器，自然语言描述任务 → 可审查 Agent Team Plan → 调度执行；明确不做通用 DAG 引擎、不做递归 spawn、不做 A2A 协议
5. **跨会话层级化 Memory（§3.5）**：在现有 `build_memory/search_memory` 基础链路上构建 User/Role/Org 三层记忆 + 写入双通道 + 治理与遗忘机制

**当前最大的研究缺口**（每条对应一个或多个 design goal）：

1. **多智能体协调机制**（goals 3, 4）—— Composer + Shared Workspace 与学术上 OneFlow/MAST/GoA 的对照验证；企业长链任务下 MAST FC1/FC2/FC3 权重的实测；BKN 共享 Memory 对 FC2 的缓解程度
2. **长链规划退化**（goals 1, 4）—— Plan-Execute 模板与 FLARE 前瞻规划的工程化结合点；MCTS 引入延迟在企业场景的可接受度
3. **工具幻觉的语义解耦防御**（贯穿 goals 3, 5）—— BKN 机制在 5k–10k 规模工具集下的有效性边界；与 [5] 类型化召回 + [7] 主动请求改写的工程化叠加
4. **常驻 Runtime 的学术对照**（goal 2）—— Heartbeat 驱动的"主动 Agent" 与现有"被动 Agent" 范式的差异；Reflection 能力的设计依据；与 OpenClaw 的边界判据
5. **跨会话记忆的层级化设计**（goal 5）—— User/Role/Org 三层记忆的合并/冲突/衰减算法；与 ContextLoader 的体系化融合

**成功标准**：
- 为 Decision Agent 0.8 路线图的 5 个增强能力提供可操作的学术支撑或反证
- 识别每个能力与业界最优实践的具体差距，并在 report.md 演化出对应的 prototype 设计章节（对齐 research-agent-triage 的 §4.2 Triage Agent prototype 设计）
- 维护 thesis 的"可证伪点追踪"，避免论点被新论文证伪后未被记录

## Taste

- **偏好治理深度直接对标的研究**——任何"BKN 同构""ISF 同构""TraceAI 同构"的论文都比纯 prompt 调优有更高优先级
- **偏好有明确 Plan-Reason-Act 分层和可观测性的研究**，而非端到端黑盒方法——Decision Agent 全链路可审计要求每层输出必须独立可追溯
- **偏好在真实多步工具调用任务中验证的工作**，而非单轮 QA 或玩具环境——Decision Agent 的核心价值在多步 AutoFlow 闭环
- **偏好有工具检索/召回机制分析的研究**——ContextLoader 的核心挑战是 5k–10k 工具集的精准按需加载
- **偏好有协同失真量化的多智能体工作**——Shared Workspace 设计需要机制级量化数据
- **偏好有"主动 Agent / 7×24 / 心跳 / Reflection"机制的研究**——Heartbeat+Cron 是 0.8 关键能力，相关学术工作稀缺，遇到必须收
- **偏好层级化记忆 / 跨会话沉淀 / 经验继承的研究**——直接服务 Memory 增强能力

## Anti-patterns

- **单一 LLM 能力提升论文（更大模型、更好 prompt）**——Decision Agent 的瓶颈是治理深度与协同架构，不是单 agent 推理能力
- **仅在游戏/形式化状态空间环境验证的规划方法**——Decision Agent 工作在非确定性企业数据场景，确定性状态转移假设不适用
- **不涉及上下文管理的多智能体协同研究**——上下文窗口是 Decision Agent 最核心的工程约束
- **纯基准测评没有机制分析**——KWeaver 需要可移植的机制（召回策略、调度算法、记忆合并算法），不是不可迁移的 benchmark 数字
- **"靠 prompt 解决工具幻觉"的工作**——BKN 语义结构化解耦才是 Decision Agent 采用的路径
- **追求通用 DAG / 通用 A2A 协议 / 端到端全自动编排的研究**——明确非目标（references/03 §四），与 LangGraph/CrewAI 类通用框架定位冲突
- **依赖自研/蒸馏模型作为前置假设的方法**——本轮迭代非目标，外部模型能力够用

## Examples

好的收录（参照）：
- `notes/01_co_evolving_llm_decision_and_skill.md` — 技能库协同进化机制对应 Memory 增强 + ContextLoader 工具管理
- `notes/03_why_reasoning_fails_to_plan_a.md` — 步进式推理结构性局限的严格形式化，直接挑战 Dolphin 状态机规划假设
- `notes/05_agent_as_a_graph_knowledge_graph.md` — 类型化召回与 BKN 同构，首份工程级证据
- `notes/06_why_do_multi_agent_llm_systems.md` — MAST 失败模式分类，Shared Workspace 设计的诊断骨架
- `notes/07_mcp_zero_active_tool_discovery_for.md` — 主动工具检索 + 模型主笔请求，揭示强模型 baseline 饱和边界

待收录优先级高的类型（与 5 个 design goals 对应）：
- 工具检索精确召回（向量 vs 语义结构化对比，5k+ 规模）
- 多智能体共享工作区机制 / 信息保真度量化
- LLM agent 在企业真实任务中的长链规划失败分析
- 主动 Agent / Always-on Runtime / Reflection 机制（goal 2 当前学术稀缺——但稀缺不默认等于机会，见可证伪点追踪表）
- 跨会话层级化 Memory / 经验继承机制（goal 5）
- Composer 类多智能体任务拆解 + Agent 选择算法（goal 4）

## 可证伪点追踪（每次 run 后由人工更新）

| 论点 | 触发证伪的发现 | 当前状态 |
|---|---|---|
| Harness-first 垂直整合是核心差异化 | 若 Dify/Coze 类浅层编排在强监管场景获得显著付费 | 待观察 |
| Shared Workspace 能显著降低 FC2 协同失真 | 若学术工作显示共享存储反而加重协调成本 | 未验证 |
| Composer 的"已有能力优先 + 半自动"路径 | 若有论文证明端到端自动 spawn 在企业场景胜出 | 未验证 |
| Heartbeat + Cron 是必要 Runtime 能力 | 若发现"主动 Agent" 在企业场景反而引入治理风险 > 收益 | 待观察 |
| Goal 2 主动 Runtime 与 Harness-first 治理可正交协同 | [8] HSC 已暴露结构性张力：Dream Mode/intrinsic goal 违反 action provenance 约束。若 ≥1 起企业治理事件由"自主认知活动"触发，goal 2 必须降级为"建议生成"层 | 已部分修订 thesis（加入"治理边界设计"约束） |
| 学术稀缺 = 机会窗口（goal 2 的 default 假设）| 已被 [8] 部分证伪——更可能"问题在企业场景下可由 cron + ReAct + 治理围栏解决"。若朴素 baseline（如固定 cron + 启发式触发 ReAct）在企业 7×24 用户调研中满足 ≥50% 真实需求，则可学习调度无优先级 | 已修订为"稀缺需要解释，不默认机会信号" |
| Goal 2 客户真实需求是"主动认知活动"而非"定时任务调度" | 若 ≥50% "7×24 数字员工"客户访谈实际只需要 cron + 失败重试，则 goal 2 应从增强能力降级为运维能力 | 待真实客户访谈 |
| 强模型 + 长上下文足以替代主动工具检索 | 已被部分支持（[7] GPT-4.1 baseline 饱和） | 已部分修订 thesis |
| BKN 语义解耦在 5k+ 工具规模仍有效 | 自家 BKN 工具集复测 Recall@5 < 拼接基线 | 未验证 |
| Decision Agent 可独立闭环不依赖 OpenClaw | 若长链任务实测发现 >30% 规划决策依赖外部长期上下文 | 待企业场景实测 |
