# Decision Agent: Multi-Agent Workers 工程问题地图与下一阶段研究优先级

> **Version:** v3.0 (6 papers, 重写)
> **Last Updated:** 2026-04-30
> **Papers:** [01](notes/01_co_evolving_llm_decision_and_skill.md) COS-PLAY · [02](notes/02_graph_of_agents_a_graph_based.md) GoA · [03](notes/03_why_reasoning_fails_to_plan_a.md) FLARE · [04](notes/04_rethinking_the_value_of_multi_agent.md) OneFlow · [05](notes/05_agent_as_a_graph_knowledge_graph.md) Agent-as-a-Graph · [06](notes/06_why_do_multi_agent_llm_systems.md) MAST
> **Thesis:** [.researcher/thesis.md](.researcher/thesis.md)

> **形式：** 内部技术报告 / Position Paper
> **作者：** xupeng
> **基于：**
> - 6 篇深读论文（COS-PLAY、GoA、FLARE、OneFlow、Agent-as-a-Graph、MAST）— 索引见 [`notes/`](notes/)
> - **Decision Agent 项目资料**（脱敏后入 [`references/`](references/)）：产品特性（[01](references/01_decision_agent_features.md)）、工具管理（[02](references/02_decision_agent_tool_management.md)）
>
> **目标读者：** Decision Agent / KWeaver / Dolphin 编排引擎 设计与工程团队
> **目标：** 把学术研究映射到 Decision Agent 当前 5 大痛点（来自 references/01）的具体差距，为**下一阶段架构决策与研究优先级**提供可操作输入

---

## 0. v1 → v6 的关键校准

> **每次 `researcher run` 累加的论文都会修正或证伪 thesis 的某个判断。本节把 6 轮校准的"已被证伪 / 修正 / 支持"汇总——避免历史噪音被新读者误读。**

| 原始判断（thesis v0 / 项目资料） | 校准后（v1–v6 之后） | 触发论文 |
|---|---|---|
| "多智能体 workers team 是 Decision Agent 核心架构挑战" | **有条件成立**：仅当 workers 异质（不同基础 LLM 或独立 LoRA）；同质工作流下 OneFlow 证明单 agent 等价且成本降低 5–10× | [4] OneFlow |
| "workers 协同失真缺乏学术层面的系统性验证" | **过时**：MAST 已用 1642 traces × 7 MAS 给出 14 类失败模式分类（FC2 协同失真 32.3%，IAA κ=0.88）；Decision Agent 的差异命题应改写为"企业场景下 FC1/FC2/FC3 权重是否与开源 MAS 一致" | [6] MAST |
| "Plan-Reason-Act 状态机驱动 workers 完成长链任务" | **规划机制层缺陷已被形式化证明**：步进式推理（CoT / ReAct / Reflexion）等价于贪心策略，首步陷阱率 55.6%，恢复概率 5.4%；Dolphin 仅做状态转移机不够，需引入前瞻规划（MCTS） | [3] FLARE |
| "BKN 语义解耦在超大工具集场景下的有效性尚缺乏系统性验证" | **首份学术实证**：类型化节点 + 加权 RRF（与 BKN 逻辑/行动分离同构）相对"拼接单向量"基线 Recall@5 +14.9%；但验证规模仅 527 工具，远低于企业目标 5k–10k | [5] Agent-as-a-Graph |
| "技能管理是核心研究方向" | **保留但弱化**：技能-决策协同进化在游戏域已验证；Decision Agent 的工具是固定 BKN 注册表（非动态学习），COS-PLAY 路径与 KWeaver 工程现实正交 | [1] COS-PLAY |
| "智能体间动态路由是必需的" | **重新定位**：GoA 图式路由在单轮 QA 验证（成本 –58%）；可作为 openclaw → decision agent 的元调度参考，但**不是 Decision Agent 内部 workers team 的核心机制**（多步任务无证据） | [2] GoA |

**保留为正确的部分**：
- ContextLoader 动态按需加载（90%+ 上下文节省）作为 Decision Agent 与通用 Agent 的核心工程差异——[5] 在 527 工具规模上提供了类型化结构的实证支撑
- 全链路可观测、可审计、可评测作为产品差异化——MAST [6] 的诊断范式可直接套到 Decision Agent 自家轨迹
- BKN 作为统一语义底座抑制工具幻觉的方向——[5] 提供首份与 BKN 同构的对比实证
- "决策提效 + 知识可信 + 流程自动化 + 数据资产可用 + 风险可控" 5 大业务价值定位——6 篇论文均未挑战

---

## 1. Decision Agent 真实战略坐标

> 本节定义"研究输入服务于什么决策"。脱离这个坐标，论文综述就只是综述。

### 1.1 产品形态（references/01）

Decision Agent **不是 agentic harness，是企业核心业务的专有 Agent 平台**：BKN 语义底座 + ContextLoader 上下文按需加载 + Dolphin 编排引擎 + ISF 安全层 + Autoflow 流程编排，覆盖配置-测评-运行-可观测-优化的全生命周期。

业务定位是"专有 Agent 落地 Data+AI 在企业核心业务场景"，**不是通用 multi-agent 框架**。这意味着：
- 学术上的"多 agent 协同"研究需要先映射到"BKN-driven workers"才有意义
- 评测维度不是"benchmark 准确率"而是 references/01 §五大业务价值

### 1.2 五大技术目标的学术映射（references/01 §五大技术目标）

| 技术目标 | 学术覆盖 |
|---|---|
| 1 降低开发复杂性（语义建模 + 低代码装配）| 主要靠工程实现；学术研究弱相关 |
| 2 维护与优化自动化（Agent 全生命周期闭环）| **MAST [6]**：FC 失败模式分类 + IAA 验证可作为"运行轨迹根因定位"骨架 |
| 3 高质量上下文（精准装配 + 按需加载）| **Agent-as-a-Graph [5]**：类型化召回与 BKN "逻辑/行动分离"同构 |
| 4 多智能体协同（统一语义 + 动态调度）| **OneFlow [4] / GoA [2] / MAST FC2 [6]**：异质性审计 + 调度参考 + 失败诊断 |
| 5 安全控制（可审计防御围栏）| 无直接论文；可观测性范式可借 MAST IAA 做基础 |

### 1.3 五大业务价值的学术不可证伪性

references/01 §五大业务价值（决策提效 / 知识可信 / 流程自动化 / 数据资产可用 / 风险可控）是产品价值主张，**不在 6 篇论文的可证伪范围内**——它们由企业落地数据（不是公开 benchmark）证明或证伪。研究输入只能服务于"实现这 5 个价值的工程机制选择"，无法直接背书价值主张本身。

---

## 2. 当前差距：从 references/01 痛点到学术映射

> references/01 §当前核心痛点列出 5 个待研究支撑的设计缺口。本节把每个痛点对照到 6 篇论文，明确"哪些已被覆盖、哪些仍是开放空白"。

| # | 项目资料认定的痛点 | 学术覆盖度 | 关键论文 | 仍开放的部分 |
|---|---|---|---|---|
| 1 | **多智能体协同失真**：Agent 间格式不统一，多轮传递信息压缩、丢约束 | 高（通用 MAS）| MAST [6] FC2 32.3%，14 个 FM 中 FM-2.4 信息扣留、FM-2.6 推理-动作错位 13.2% 直接对应 | 企业长链任务 FC2 实测占比未知；BKN 共享 Memory 是否真的降低 FC2 缺自证 |
| 2 | **全局调度/仲裁缺失**：缺冲突治理，多 Agent 易循环或重复劳动 | 低 | MAST FM-1.3 步骤重复 15.7% 给出"循环"诊断信号；GoA [2] 提供推理时图路由参考 | Dolphin 状态机的仲裁机制无对标；多步任务的"冲突治理"无学术证据 |
| 3 | **Worker 独立闭环能力**：是否需持续依赖 Dolphin/openclaw 上下文 | 中 | COS-PLAY [1] 技能库智能体跨 episode 无状态运行（间接支持）；FLARE [3] 假设规划信号 r̂ 可用，若 r̂ 只在 manager 持有则反证依赖 | 企业真实任务上的 worker 独立性未测 |
| 4 | **工具幻觉**：模型越强越严重，BKN 语义解耦待验证 | 中 | Agent-as-a-Graph [5] 类型化节点 +14.9% Recall@5 提供首份与 BKN 同构的实证 | 527 工具远低于企业 5k–10k；KWeaver 自家工具集复测未做；类型权重 αA:αT=1.5:1 在 BKN 数据上是否仍最优未知 |
| 5 | **长链规划退化**：Plan-Reason-Act 在 >5 步任务质量缺基准 | 高（理论）| FLARE [3] 形式化证明步进式推理结构性不足；前瞻 MCTS 是修复路径 | KGQA / 玩具环境到企业 AutoFlow 的距离未测；MCTS 引入的延迟代价在企业场景的可接受度未知 |

**两类缺口**：
- **学术已覆盖但企业场景未验证**（痛点 1、4、5）→ 工程团队可主导内部复测
- **学术弱覆盖**（痛点 2、3）→ 需主动寻找新工作或自行做研究

---

## 3. 学术研究按"工程缺口"重新组织

> 不是论文综述，而是把 6 篇论文重组到上面 5 个痛点 + 1 个先决问题。每节末尾给出对 Decision Agent 的可操作启示。

### 3.1 §0 先决：多智能体结构是否必要？（OneFlow）

[4] OneFlow 形式化证明（Proposition 1）：在确定性工具副作用、可见历史路由、共享随机性下，单 LLM 模拟器与多智能体系统轨迹分布完全等价。OneFlow 单智能体在 6 个公开基准上匹配或超越 AFlow 多智能体，成本降低 5–10×（HumanEval $0.020 vs. $0.198）。

**对 Decision Agent 的强约束**：[4] 等价结论的前提是"同质工作流"（|B(W)| = 1，所有 agents 共享同一基础 LLM）。Decision Agent 的 BKN/ContextLoader/Dolphin/ISF 是否构成异质配置（不同基础模型或 LoRA），目前无内部审计结果。

两条决策路径——
- **同质**：架构优先级应从"多智能体协同机制"转向"单智能体工作流设计 + 前瞻规划"
- **异质**：论题前提保留，多智能体协同仍是核心挑战

**这是优先级 #0 的工程决策**，先于其他研究方向。详见 §4 P0-1。

### 3.2 痛点 1 + 2（协同失真 + 调度仲裁）→ MAST 范式 + GoA 路由

[6] MAST 把"协同失真"从直觉变成可度量：FC2 32.3%，14 个 FM 中 FM-2.6 推理-动作错位 13.2%、FM-2.4 信息扣留、FM-2.5 输入忽略、FM-2.3 任务偏离 7.4%——直接对应 references/01 痛点 1 的"格式不统一 / 信息丢约束"症状。FM-1.3 步骤重复 15.7% 是 references/01 痛点 2 的"循环 / 重复劳动"可观测代理。

[2] GoA 在推理时构图路由：节点采样 + 边采样 + 双向消息传递 + 图池化。GoAMax 用 3 agents 超越 MoA 的 6 agents，成本降低 58%——但仅在单轮 QA 验证。

**对 Decision Agent 的可操作启示**：
- **P0**：在 BKN+ContextLoader+Dolphin+ISF 真实业务轨迹上跑自家版 MAST（详见 §4 P0-2）
- **P1**：FM-1.3 自家轨迹占比是 Dolphin 仲裁机制是否到位的可观测代理
- GoA 的图路由不是 Decision Agent 内部机制，但**可作为 openclaw → decision agent 的元调度参考**——前提是 openclaw 工程化时确实需要"按 query 动态选 worker"

### 3.3 痛点 3（Worker 独立闭环）→ COS-PLAY 间接支持 + FLARE 反证维度

[1] COS-PLAY 的技能库智能体跨 episode 无状态运行，对论题"workers 不需依附长期助理"提供**初步支持**。但 COS-PLAY 在游戏域，Decision Agent 的工具是固定 BKN 注册表（不动态学习），路径正交——支持是间接的。

[3] FLARE 假设规划时可用评估信号 r̂——若 Decision Agent worker 的 r̂ 只能由 openclaw 持有的长期上下文计算，则 worker 对 manager 的依赖远超"调度入口"。FLARE 没探讨这点，但**其假设暴露了一个反证维度**：若内部规划必须依赖外部上下文，独立闭环不成立。

**操作建议**（详见 §4 P1-2）：审计 Decision Agent worker 的规划决策依赖项——若全部可由 BKN + 当前 trace 计算，独立闭环成立；若需外部对话历史或 manager 状态，需要重新评估分层架构。

### 3.4 痛点 4（工具幻觉）→ Agent-as-a-Graph 首份实证

[5] Agent-as-a-Graph 在 LiveMCPBench（527 工具）上把工具/agent 建为二部图两类节点 + 加权 RRF 检索：Recall@5 = 0.85 vs. ScaleMCP 的 0.74（+14.9%）。**这 14.9% 不是更好的 embedding，而是把"逻辑（agent 描述）"和"行动（工具描述）"从同一向量拆出来——结构上与 references/02 §方案二（BKN 驱动语义精准执行）完全同构**。

最优类型权重 αA:αT = 1.5:1（轻度偏向 agent 覆盖）；架构无关（跨 8 种 embedding 模型 std≈0.02）。

**对 Decision Agent 的可操作启示**：
- **P0**：KWeaver 自家工具集（构建 5k+ 规模）复测类型化结构 vs. 拼接基线（详见 §4 P0-3）
- **P1**：暴露 αA:αT 作为 ContextLoader 的可调旋钮——查询意图模糊时偏向父技能/父域覆盖（αA 高），意图明确时偏向工具级精准（αT 高）
- **缺口**：[5] 不报 wall-clock 延迟也不分析图遍历成本，5k–10k 工具下是否仍可接受需自家测

### 3.5 痛点 5（长链规划退化）→ FLARE 前瞻 MCTS

[3] FLARE 形式化证明步进式推理（CoT / ReAct / Reflexion）等价于最大化局部代理分数的贪心策略，**结构性局限**而非工程缺陷：第一步决策中贪心策略 55.6% 选中陷阱动作，首步错误后单步恢复概率仅 5.4%。LLaMA-8B + FLARE 频繁超越 GPT-4o + 标准推理——规划能力无法通过参数规模弥补。

[6] MAST 在执行层经验性佐证：FM-1.5 终止条件无感知 12.4% + FM-3.1 过早终止 6.2% = 18.6% 失败属于"规划深度不足"。

**对 Decision Agent 的可操作启示**：
- **P1**：FLARE 前瞻 MCTS 应作为 Dolphin 在长链 AutoFlow 中的规划骨架，而不仅仅是状态转移机（详见 §4 P1-1）
- **缺口**：FLARE 假设确定性状态转移、显式状态、规划时可用评估信号——三条假设在企业开放任务上均可能违背

### 3.6 横切诊断：MAST 范式作为 Decision Agent 可观测性 SLA

MAST [6] 不只回答"协同失真"，也提供**可观测性的标准化范式**：
- 14 个 FM × 3 FC 分类法：可作为 Decision Agent trace 的标签集合
- Cohen's κ=0.88 IAA / 0.79 域外验证：作为内部失败标注的质量门槛
- LLM-as-Judge κ=0.77：把 Decision Agent 的可观测性从"trace 收集"升级到"trace 自动诊断"
- 同模型不同架构差异显著（同 GPT-4o，MetaGPT vs. ChatDev FC1/FC2 失败差 60–68%，FC3 差 1.56×）：架构选择 > 模型选择，且不存在全方位最优——Dolphin 设计取舍直接指向"显式审阅 → 减 FC1/FC2 但增 FC3"，ISF 安全层若加显式校验需关注 FC3 副作用

---

## 4. 直接服务当前差距的具体建议

按工程优先级排列：**P0 = 立即开始，P1 = 下个迭代，P2 = 季度规划**。

### P0-1 异质性审计（先决问题）
**动机**：[4] OneFlow 等价结论的前提决定"多智能体架构是否仍是核心挑战"。
**操作**：审 BKN/ContextLoader/Dolphin/ISF 的基础模型与适配器，确认是否满足 |B(W)| > 1（不同 base / 不同 LoRA / 不同温度的判定标准需先内部约定）。
**输出**：一份内部 RFC，标记每个 worker 的基础模型来源 + 是否独立 LoRA。
**判定**：若同质，启动"单智能体工作流 + FLARE 前瞻规划"研究方向；若异质，进入 P0-2 + P0-3。

### P0-2 自家版 MAST 度量
**动机**：[6] MAST 的 44/32/24 是开源 MAS 参考点，不是 Decision Agent 目标——企业长链任务的 FC 分布很可能显著偏离。
**操作**：抽取 N≥150 条真实业务轨迹（混合长链任务 + 工具调用），3 名内部专家用 MAST 14 FM 标注，做 IAA 直到 Cohen's κ ≥ 0.85；然后用 LLM-as-Judge 标注其余轨迹。
**输出**：Decision Agent 自家 FC1/FC2/FC3 分布 + 14 个 FM 占比；与 MAST 公开数字对比表。
**判定**：FC2 < 5% 表示协同失真已被工程化解决，论题该成分被部分证伪；FC2 > 32% 表示论题前提强保留并应将 14 个 FM 作为可观测性 SLA 对标基线。

### P0-3 类型化召回工具集复测
**动机**：[5] 的 +14.9% Recall@5 在 KWeaver 自家工具集是否可迁移决定 BKN 设计的实证支撑强度（references/02 §方案二的有效性边界）。
**操作**：构建 5k+ 规模 KWeaver 工具集 benchmark；分别用"类型化节点 + 加权 RRF（αA:αT=1.5:1）"和"拼接单向量"基线测 Recall@5 / nDCG@5；并扫描 αA:αT 在自家数据上的最优值。
**输出**：BKN vs. ScaleMCP 风格基线对比表 + 最优 αA:αT 自家扫描结果 + wall-clock 延迟（[5] 未报）。
**判定**：若 +Recall@5 < 5%，BKN 实证支撑被削弱，需寻找其他防御机制；若 ≥ 14.9%，BKN 设计获得跨数据集可重复证据，下一步把 αA:αT 暴露为 ContextLoader API。

### P1-1 Dolphin 状态机的规划层升级
**动机**：[3] FLARE FM-1.5/FM-3.1 共 18.6% 是直接动机；当前 Dolphin 是状态转移机不是规划器。
**操作**：在长链 AutoFlow 上引入 FLARE 风格前瞻 MCTS + 后向价值传播 + 轨迹记忆，对照原状态机度量首步陷阱率。
**输出**：长链任务上的"规划质量 vs. 延迟"曲线；判定企业可接受的延迟代价上界。

### P1-2 Worker 独立性边界审计
**动机**：references/01 痛点 #3 + FLARE [3] 假设暴露的反证维度。
**操作**：选 N 个具代表性长链任务，标注每步规划决策依赖的输入来源（BKN / 当前 trace / openclaw 长期上下文）。
**输出**：决策依赖热图——若长期上下文依赖 > 30%，分层架构需重新评估，二者耦合程度更接近"持续上下文依赖"而非"调度入口"。

### P2 ContextLoader 类型权重旋钮 + 调度元层
**动机**：[5] 暴露 αA:αT 作为可调；[2] GoA 提供 openclaw → decision agent 的元调度参考。
**操作**：先做 P0-3 复测确认数字可迁移，再把 αA:αT 暴露为 BKN 检索 API 参数；调度元层作为 openclaw 工程化时的备选实现。

---

## 5. 阻塞与限制

> 6 篇论文都验证不了的事情，需要 Decision Agent 团队自身回答。

1. **企业长链任务上的规划-延迟权衡**：FLARE 前瞻 MCTS 在 KGQA / ALFWorld 验证；企业 AutoFlow 涉及外部审批门、延迟奖励、动态工具集——三者在论文中均未出现。
2. **闭源 / 商业 MAS 的失败分布**：MAST [6] 明确排除 Manus 等闭源系统；Decision Agent 作为商业平台，其 FC 分布与开源 MAS 是否一致**没有任何直接证据**。
3. **企业工具集 5k–10k 规模的检索延迟**：Agent-as-a-Graph [5] 在 527 工具量级验证，不报 wall-clock；ContextLoader 真正起作用的量级需自测。
4. **异质性的边界条件**：OneFlow [4] 区分同质 vs. 异质，但"加 LoRA"是否构成异质未明确——是 base model 完全不同？是不同 LoRA 头？是不同温度？P0-1 审计前需先内部约定判定标准。
5. **manager-worker 分层在企业场景的实证**：thesis 核心部署主张（decision agent + openclaw）目前只有间接支持证据（COS-PLAY 技能库无状态运行），无端到端实证。
6. **MAST FC3 的解释循环**：MetaGPT/ChatDev 总失败少但 FC3 占比高——校验器把 FC1/FC2 重定向到 FC3 是架构定义的副产品，不是干净的根因信号。Decision Agent 的 ISF 安全层若加入显式校验，可能复制这一副作用。

---

## 6. 一句话结论

**Decision Agent 当前最重要的工程决策是异质性审计（P0-1）——它决定了多智能体架构是否仍是核心挑战；其次是自家版 MAST 度量（P0-2）和 BKN 类型化召回复测（P0-3），三者合并后才能给出"下一阶段架构优先级是协同还是规划"的可证伪回答。在此之前，所有"workers 协同 / 长链规划 / 工具幻觉"的工程投入都是基于未验证假设的赌注。**

---

## 附录 A：Decision Agent 组件 ↔ 论文术语速查

| Decision Agent | 论文术语 | 对应工作 |
|---|---|---|
| BKN（业务知识网络）| 类型化节点 + 显式关系建模 | Agent-as-a-Graph [5] 的二部图（agent / tool 双类型节点） |
| ContextLoader（动态加载）| dynamic / lazy tool retrieval | [5] sources distribution: 39.13% agent + 34.44% tool via owner edge |
| Dolphin 编排引擎 | MAS framework / orchestrator | MAST [6] 评测的 7 个开源 MAS（ChatDev、MetaGPT、HyperAgent…）|
| Plan-Reason-Act 状态机 | step-wise reasoning（CoT / ReAct）| FLARE [3] 形式化为贪心策略 |
| 前瞻规划 | lookahead / MCTS planning | FLARE [3] 的 MCTS + 后向价值传播 |
| Workers team | multi-agent system | OneFlow [4]（同质等价）+ MAST [6]（失败分类） |
| 协同失真 | inter-agent misalignment | MAST [6] FC2 类（FM-2.1 ~ FM-2.6）|
| 工具幻觉 | tool hallucination | references/02 §1；[5] 缓解 via 类型化召回 |
| ISF 安全层 + Human-in-the-loop | explicit verification | MAST [6] FC3 类 + Insight 3（"verifier presence 不充分"）|
| Autoflow 流程编排 | workflow / multi-step task | OneFlow [4] AFlow 比较场景 |

---

## 附录 B：可证伪点追踪

| # | 可证伪声明 | 证伪路径 | 触发论文 |
|---|---|---|---|
| 1 | 多智能体 workers team 是 Decision Agent 核心架构挑战 | 异质性审计：若同质（\|B(W)\|=1）则部分证伪 | OneFlow [4] |
| 2 | BKN 逻辑/行动分离比拼接单向量更精准 | KWeaver 自家工具集复测：若 +Recall@5 < 5% 则证据被削弱 | Agent-as-a-Graph [5] |
| 3 | workers 协同失真是 Decision Agent 关键瓶颈 | 自家 FC2 度量：若 < 5% 则部分证伪（已工程化解决）；若 > 32% 则强保留 | MAST [6] |
| 4 | workers 不需依附长期助理上下文即可闭环 | 决策依赖热图：若长期上下文依赖 > 30% 则分层架构需重新评估 | FLARE [3] / COS-PLAY [1] 间接 |
| 5 | 步进式 Plan-Reason-Act 在长链任务足够 | FLARE 已形式化证伪（首步陷阱率 55.6%、恢复率 5.4%）；自家长链任务延迟代价测试可决定是否引入 MCTS | FLARE [3] / MAST FM-1.5 [6] |

---

## 附录 C：参考资料

**论文**（详见 [`notes/`](notes/)）：
- [01] COS-PLAY: Co-Evolving LLM Decision and Skill Bank Agents (Wu et al., 2026)
- [02] GoA: Graph-of-Agents (Yun et al., ICLR 2026)
- [03] FLARE: Future-Aware Lookahead via MCTS (Wang et al., 2026)
- [04] OneFlow: Single-Agent Sufficiency Baseline (Xu et al., 2026)
- [05] Agent-as-a-Graph: Knowledge Graph for Tool/Agent Retrieval (Nizar et al., 2025)
- [06] MAST: Multi-Agent Systems Failure Taxonomy (Cemri et al., 2025)

**项目资料**（脱敏后入 [`references/`](references/)）：
- [01] Decision Agent 特性梳理（产品 PPT 来源）
- [02] Decision Agent 工具管理（产品 PPT 来源）

**待补**：
- 闭源 / 商业 MAS 的失败模式评测（如有）
- ContextLoader 5k+ 规模工具检索的现有学术 benchmark

---

## 版本更新日志

| 版本 | 日期 | 关键变化 |
|---|---|---|
| v1 | 2026-04-27 | [01] COS-PLAY 加入；建立技能库协同进化基线 |
| v2 | 2026-04-27 | [02] GoA 加入；推理时图路由参考 |
| v3 | 2026-04-27 | [03] FLARE 加入；步进式推理结构性局限诊断 |
| v4 | 2026-04-27 | [04] OneFlow 加入；多智能体必要性挑战 |
| v5 | 2026-04-29 | [05] Agent-as-a-Graph 加入；BKN 语义解耦首份学术实证 |
| v5.1 | 2026-04-30 | [06] MAST 加入（PR #9）；横切诊断维度 |
| **v3.0 重写** | 2026-04-30 | **重构为 Position Paper 形态**：新增 §0 v1→v6 校准、§1 战略坐标、§2 痛点-论文映射；§3 按工程缺口而非论文顺序组织；§4 按工程优先级排列建议；新增 §5 阻塞与限制、§6 一句话结论；附录 B 可证伪点追踪 |
