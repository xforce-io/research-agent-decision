# Decision Agent: Research Report

> **Version:** v9 (9 papers)
> **Last Updated:** 2026-05-02
> **Papers:** [01](notes/01_co_evolving_llm_decision_and_skill.md), [02](notes/02_graph_of_agents_a_graph_based.md), [03](notes/03_why_reasoning_fails_to_plan_a.md), [04](notes/04_rethinking_the_value_of_multi_agent.md), [05](notes/05_agent_as_a_graph_knowledge_graph.md), [06](notes/06_why_do_multi_agent_llm_systems.md), [07](notes/07_mcp_zero_active_tool_discovery_for.md), [08](notes/08_simulating_human_cognition_heartbeat_driven_autonomous.md), [09](notes/09_llm_based_multi_agent_blackboard_system.md)
> **Thesis:** [.researcher/thesis.md](.researcher/thesis.md)

---

## 步进式推理不够用：从局部最优到结构性失败

最锐利的发现来自 FLARE [3]：步进式 LLM 推理（CoT、ReAct、Reflexion）在形式上等价于最大化局部代理分数的贪心策略。这不是工程缺陷，而是结构性局限——Proposition 3.2/3.3 证明：对任意固定 beam width B，存在使最优轨迹在深度 1 被不可逆剪枝的环境；而 k=1 前瞻在最坏情况下严格优于任意步进策略。

实证数据强化了这一结论：第一步决策中，贪心策略在 55.6% 的情况下选中"陷阱"动作，首步错误后单步恢复概率仅 5.4%。LLaMA-8B + FLARE 频繁超越 GPT-4o + 标准推理 [3: §5.2]——规划能力无法通过参数规模弥补。

这对论题的含义直接：若 decision agent workers 内部采用步进式推理（当前绝大多数实现），在需要超过若干步序列决策的任务上将系统性失败。Workers team 的协同不能掩盖单个 worker 规划机制的根本缺陷。

**关键局限**：FLARE 的实验场景（KGQA 图遍历、ALFWorld 玩具环境）假设确定性状态转移、显式状态表示、规划时可用评估信号——三条假设在开放式 agentic 任务中均可能违背。从图遍历"长时序"到实际多步工具调用工作流的距离尚未测量 [3: Weaknesses]。

---

## 技能管理：知道做什么 vs. 如何选择

COS-PLAY [1] 解决的问题与 FLARE 正交：不是"如何在已知动作中选择"，而是"如何构建可复用的动作抽象"。将决策智能体（A_D）与技能库智能体（A_S）解耦并用 GRPO 联合训练，7–25 轮协同进化后：8B 模型在四款单人游戏上平均奖励超 GPT-5.4 25.1%；每技能平均被实例化 12–49 次，2–6 次合约精化 [1: §5.1, §5.3]。

关键设计发现：技能库分布与决策策略分布必须对齐，错位版本反而降低性能 [1: §5.2]。这意味着技能库不是静态知识库，而是与决策策略共同演化的动态伙伴。技能库智能体跨 episode 无状态运行，为决策智能体独立部署提供支持——但验证域局限于游戏环境。

**与 FLARE 的组合空间**：COS-PLAY 的 A_D 使用 GRPO 训练的策略（规划嵌入权重），FLARE 明确区分"离线 RL 无法在线修订"与"前瞻规划"的差异。若将 FLARE 的前瞻机制叠加到 COS-PLAY 的技能库框架上，理论上可同时获得技能复用和规划深度——但两篇论文均未触及这一组合，属于公开研究缺口。

---

## 推理时路由：多智能体协同的低成本起点

GoA [2] 在不同系统层级上回答了另一个问题：推理时如何以最低成本将查询路由到最相关的专家子智能体？四阶段执行（节点采样→边采样→双向消息传递→图池化），GoAMax 用 3 个 agents 在 MMLU、MMLU-Pro、MedMCQA 上超越 MoA 的 6 个 agents，推理成本降低约 58% [2: §4.2, Table 2]。

GoA 的贡献对论题的映射是间接的：它验证了"元 LLM 按查询动态路由至专家子智能体"的可行性，可作为 openclaw→decision agent 调度链的轻量实现参考。但其验证场景（单轮多域 QA）与多步任务执行的距离和 FLARE 的 KGQA 场景一样大——GoA 的"专家子智能体"是单轮推理者，不是能执行多步工作流的 decision agent workers。

---

## 多智能体结构是否必要？同质工作流的挑战

[4] 对论题的冲击最直接。OneFlow 形式化证明：在确定性工具副作用、可见历史路由、共享随机性三个条件下，单 LLM 模拟器与多智能体系统的轨迹分布完全等价（Proposition 1）[4: §3.1]。实证结果紧随其后：OneFlow 单智能体在 6 个公开基准上匹配或超越 AFlow 多智能体，成本降低 5–10×（HumanEval: $0.020 vs. $0.198）[4: §4.2.1, Table 1–2]。TravelPlanner 上，单智能体 Pareto 优于多智能体（精度更高，成本更低）[4: Fig. 3]。

这对论题的挑战是有条件的。[4] 的等价结论有一个关键前提：**同质工作流**（|B(W)| = 1，所有 agents 共享同一基础 LLM）。这正是 Decision Agent 架构需要首先回答的问题：

- 如果 Decision Agent 的 workers 本质上只是同一 LLM 加不同 prompt（同质配置），[4] 意味着多智能体调度开销毫无必要——单智能体顺序执行即可。
- 如果 BKN 语义层、ContextLoader 动态加载、ISF 安全层、Dolphin 状态机各自引入了不同的基础模型或专化 LoRA 适配器（异质配置），[4] 的结论不适用，多智能体协同仍然必要。

[4] 自己提供了分界线：COS-PLAY [1] 使用独立 LoRA 适配器的异质双智能体，明确落在"单 LLM 模拟无法捕获"的场景 [4: §6]。但对于 Decision Agent，这个异质性是否为真，目前尚无实证回答。

**异质性不是默认成立的工程假设。** 在确认 Decision Agent 属于异质配置之前，"多智能体 workers team 是核心架构"这一论题前提是未经检验的。[4] 将"证明多智能体必要性"从隐含假设变成了需要主动回答的问题。

**[4] 的关键局限**：所有实验 temperature=0（等价结论不覆盖随机采样）；最复杂基准为 TravelPlanner（单会话规划，非多会话企业工作流）；异质场景实验标记为"pilot"，未充分优化；并行 fan-out-fan-in 工作流未覆盖 [4: §Weaknesses]。

---

## 调用前的检索层：BKN 语义解耦的首份实证

Decision Agent 的产品文档把"工具检索/召回"列为最大的工程瓶颈：企业可能有数千工具，全量加载不现实，BKN 通过"逻辑（业务意图）/ 行动（具体执行）"显式分离 + 业务对象关联来抑制工具幻觉 [references/02 §方案二]。Agent-as-a-Graph [5] 在学术层面给出了与 BKN 同构的设计选择的直接对比实验。

在 LiveMCPBench（70 服务器/527 工具/95 问题）上，[5] 把工具与父 agent 建为二部图的两类节点，分别在统一向量空间 top-N 检索，再用类型加权 RRF（αT/(k+r) 用于工具节点，αA/(k+r) 用于 agent 节点）重排，最后沿 owner 边遍历至 K 个唯一 agent。Recall@5 = 0.85、nDCG@5 = 0.47，相对前 SOTA ScaleMCP（"拼接单向量"基线）+14.9%/+14.6% [5: §4.2, Table 1]。

**这 14.9% 的来源很重要**：它不是更好的 embedding 或更大的索引，而是**把工具描述和 agent 描述从同一向量中拆出来变成显式的两类节点**。这与 BKN 的"逻辑/行动分离"在结构上同构——把语义不同的两层关注点从"拼接成一段文本"重构为"分别建模并通过显式关系连接"。论题原本将 BKN 列为"主要防御机制"但承认"在超大工具集场景下的有效性尚缺乏系统性验证"——[5] 在 527 工具量级上首次给出了这一架构选择的实证证据。

**类型权重的可调性是一个工程信号**。最优 αA:αT = 1.5:1（轻度偏向 agent 覆盖）；1:1 退化到 0.83/0.46，3:1 跌至 0.76，1:3 跌至 0.80 [5: §4.4, Figure 2]。这暗示 ContextLoader 的工具选择机制可以暴露同类型的权重旋钮：当查询意图模糊时偏向父技能/父域的覆盖召回，当意图明确时偏向工具级的精准召回。这是一个比"调一组超参"更具体的设计建议。

**架构无关性是迁移性的强信号**。跨 8 种 embedding 模型（Vertex/Gemini/Titan v1-v2/OpenAI ada/3-小/3-大/MiniLM）平均 Recall@5 提升 19.4%，std≈0.02 [5: §4.3, Table 2]——这意味着无论 ContextLoader 选哪种 embedding 后端，类型化结构本身的收益都可以预期。

**两个明显的工程缺口**：
1. **规模差距**。LiveMCPBench 527 工具远低于论题"企业数千工具"目标，也低于 ContextLoader "支持工具规模提升 10 倍以上"声称真正起作用的量级 [references/02 §综合效果]。[5] 不报告 wall-clock 延迟也不分析图遍历成本，无法直接评估在 5k–10k 工具下是否仍可接受。
2. **超参数选择与测试集污染**。最优 αA:αT 在 LiveMCPBench 上扫描后又在同一基准上报告，没有 held-out 验证集。2.41% 的"加权 RRF 增益"是上界估计；跨域泛化未证实 [5: §Weaknesses]。

**对 Decision Agent 的可操作启示**（按工程优先级）：
- **接受类型化解耦的实证支持**——BKN "逻辑/行动分离"不再只是设计直觉，[5] 提供了直接对比数据；但需要在 KWeaver 自身工具集上复测以确认数字可迁移。
- **填补规模缺口**——这是论题明确标注的高优先级研究空白，且是一个可由 Decision Agent 团队自身在内部数据上完成的实验（构建 5k+ 规模工具集 + 类型化召回基准），其产出对学术界也有价值。
- **暴露类型权重作为可调旋钮**——而不是把"工具召回"做成一个不可观测的黑盒。

---

## MAS 失败的"地图"已经画出来了——但是开源、单会话的版本

MAST [6] 是这批论文里第一个不提架构、只做诊断的工作。它对 7 个开源 MAS（HyperAgent、AppWorld、AG2、ChatDev、MetaGPT、Magentic-One、OpenManus）在编码/数学/常识基准上跑出 1642 条执行轨迹，用扎根理论编码出 14 个失败模式，分布于 3 大类：FC1 系统设计 44.2% / FC2 智能体间错位 32.3% / FC3 任务校验 23.5% [6: §4, Figure 1]。三专家在 MAST 上 Cohen's κ=0.88，OpenAI o1 LLM-as-Judge 标注器 κ=0.77——这是该领域第一份具备可复制度量的失败分类法 [6: §3.2–3.4]。

对论题最直接的影响：thesis §1 写"workers 之间的协同失真、长链规划退化和多智能体调度仲裁，仍缺乏学术层面的系统性验证"。MAST 已经在通用 MAS 层面填补了"协同失真"这一格——FC2 占 32.3%，且其中三大头部失败模式（FM-1.3 步骤重复 15.7%、FM-2.6 推理-动作错位 13.2%、FM-1.5 终止条件无感知 12.4%）足以作为 Decision Agent Dolphin 状态机的对标基线。论题措辞应从"未度量"更新为"已有通用基线，企业多会话场景待对标"——详见 `contradictions.md §Contradiction: 协同失真验证缺口已被部分填补`。

**MAST 的两条横切发现对 Decision Agent 直接有用：**

1. **同模型不同架构差异显著**（同 GPT-4o 下，MetaGPT 比 ChatDev 少 60–68% FC1/FC2 失败，但多 1.56× FC3 失败）[6: §5.1, Figure 9]。这说明架构选择对失败模式分布的影响远超模型选择，且不存在"全方位最优架构"——加上显式审阅阶段（ChatDev）减少 FC1/FC2 但放大 FC3。Dolphin 状态机的设计取舍可以直接套这个权衡：审阅/校验阶段越显式，FC3 风险越高，因此 ISF 安全层不应只检查"是否完成"还要检查"校验是否真的有效"。
2. **MAST 指导的针对性干预收益可观**：ChatDev 角色规约修复 +9.4%，增加高层目标校验步骤 +15.6%——同模型、不改架构、只改流程就拿到两位数提升 [6: §1, Appendix H]。这意味着 Decision Agent 可以在不动 BKN/ContextLoader/Dolphin 内核的前提下，通过 MAST 风格的失败诊断 + 针对性流程修补获得显著可靠性提升。

**MAST 的关键局限对 Decision Agent 的迁移性有约束：**

- **闭源 MAS 被排除**：Manus 在 ProgramDev 上 60%，但因轨迹不透明被排除出 MAST-Data。Decision Agent 作为商业平台属于这一类——MAST 的失败分布在闭源、企业级 MAS 上是否成立，没有任何直接证据 [6: §1]。
- **共生评估的循环风险**：MAST 用同一 LLM-as-Judge 既做失败标注又做干预后效果评估，缺乏独立人工复核 [6: §3.3 局限]。Decision Agent 如果照搬这个范式，失败率改进的 9.4%/15.6% 数字需要打折读。
- **任务复杂度未拆分**：ProgramDev 显式标注为"相对简单"，MAST 没有按任务难度拆分 FC 分布 [6: §D]。Decision Agent 的企业长链任务结构性更复杂，FC1/FC2/FC3 的相对权重很可能与 MAST 报告的 44/32/24 显著偏离。
- **FC3 数据的解释循环**：带显式校验器的 MAS（MetaGPT、ChatDev）总失败更少但 FC3 占比更高——这部分是架构定义的副产品（校验器把 FC1/FC2 重定向到 FC3），不是干净的根因信号 [6: §4 FC3]。

---

## 主动工具检索的边界：强模型 baseline 已饱和

MCP-Zero [7] 与 [5] 同攻 MCP catalog 工具检索，但走相反方向：[5] 押"显式类型节点+类型加权 RRF"，[7] 押"LLM 在执行中主笔结构化请求 `<tool_assistant>{server, tool}` 触发外部检索"。两者各自给出了 BKN "逻辑/行动分离"的两条独立证据线：[5] 在 LiveMCPBench 527 工具上 Recall@5 +14.9%；[7] 在 APIBank 多轮 Full 上从用户查询直接检索的 65.05% 拉到模型主笔的 90.32%（+25 个百分点）。

但 [7] 揭示了一条对 Decision Agent 重要的边界条件：**"主动检索"对精度的杠杆主要落在弱模型上**。在 needle-in-haystack（≤2,797 工具）实验上，GPT-4.1 等强模型 baseline 已饱和——MCP-Zero 在精度上无增益，仅省 token。如果 Decision Agent 以最强基础模型落地（这是 thesis "依赖外部模型能力够用"非目标的合理前提），ContextLoader "主动检索"的实际增益更可能落在 token 成本侧而非精度侧。这要求修正一个隐含的过度承诺：BKN + 主动检索不会"提升精度"，而是"在保持精度同时显著降低上下文压力"——后者在论题"上下文窗口工程约束"的优先级下仍是核心价值。

[5] 与 [7] 在 LiveMCPBench 上有直接对照：[5] Recall@5 0.85 vs [7] 0.70——[5] 系统性击败 [7]。这暗示 Decision Agent 的工程路径应是 **[5] 的类型化检索后端 + [7] 的请求改写前端组合**，而非二选一：让 BKN 承担"逻辑/行动分离"的结构化解耦，让主笔请求改写承担"模糊查询 → 精准语义请求"的桥接。

**[7] 的关键缺陷**：APIBank Full 仅 48 unique tools，"thousands of tools" 动机与精度评测断层；§3.2 信息论框架（互信息最大化、O(n)→O(m+k)）从未被实验 instrumented，是装饰性事后合理化；未与 RAG-MCP / ScaleMCP / Tool2Vec / AnyTool 等已发表方法对比 [7: §4.1, §3.2, §5.3]。

---

## Heartbeat 调度的学术稀缺：反证案例而非正面支撑

HSC [8] 是迄今唯一直接以 "heartbeat scheduling" 为标题的 LLM agent 工作。其形式化框架（macro/micro 双层调度、Dream Mode 三大功能、状态增强 `s_t' = φ(s_t, h_t)`）正面对应 0.8 路线图 goal 2（Heartbeat + Cron + Reflection）。但论文实验完全脱离主张——§III-IV 是 LLM agent + 可插拔认知模块 + 自主学习调度，§VI 实验却是 LSTM 在合成 1,800 天 × 24 小时 × 6 类动作序列上的监督 next-action 预测：没有 LLM、没有任何认知模块的实例化、没有 Dream 触发、没有 RL 步骤、没有任何 baseline。

**这是一个反证案例而非支持案例。** 它揭示 thesis 的"学术稀缺信号"判断是真实的——稀缺到首篇相关论文的实验完全不验证主张；同时也降低了"通过文献合成获得 Heartbeat 设计指南"的预期。可吸收的语言资源（macro/micro 调度命名、Dream Mode 三功能、状态增强防重复）应作为概念框架使用，**不应被引为机制有效性证据**。

更重要的是，[8] 暴露了 thesis 治理优先核心定位与 goal 2 的结构性张力：HSC 的 Dream Mode 包含"自主目标设定"和"合成经验生成"，这在结构上违反 ISF/TraceAI 的 "action provenance must be explicit" 治理约束——主动 agent 越主动，治理越被动。thesis 已据此修订（加入"治理边界设计"约束：主动行为必须有显式调度策略来源、必须 TraceAI-first、涉及外部副作用必须经过 ISF + Human-in-the-loop），并把 "学术稀缺 = 机会窗口" 默认假设降级为"稀缺需要解释，不默认机会信号"。

**对 Decision Agent 的实践含义**：goal 2 的工程优先级从"补足空白"调整为"在治理边界内实现有限主动模式"——Reflection 类能力可保留但限定在"建议生成"层而非"自主执行"层。Heartbeat + Cron 的实证基础需要由 Decision Agent 团队自行建立。

---

## Shared Workspace 的工程化原型：Blackboard MAS 给 goal 3 / goal 4 的直接证据

Blackboard MAS [9] 是这批论文里第一篇直接对应 0.8 路线图 goal 3（Shared Workspace）与 goal 4（Composer）的工作。它把多智能体通信范式从 master–slave 替换为：主智能体（ReAct，T=10 步）以 `request_help` 写入共享 blackboard β；每个 cluster 的 file agent 与 search agent **自判**可否响应，能则写入**主智能体独占的响应板 β_r**；helper 之间互不可见。在 KramaBench / DSBench-filtered / DA-Code-filtered 合计 ~448 题 × 4 backbone（Qwen3-Coder-30B / Gemini 2.5 Flash & Pro / Claude 4 Opus）上较 Master–Slave 相对提升 13%–57%；File discovery（Gemini 2.5 Pro）F1 0.561 vs Master–Slave 0.513 vs RAG 0.247 [9: §4.2 Table 1, Table 2]。

**对 goal 3 (Shared Workspace) 的设计直接复用**：
- "β / β_r 分离 + helper 互不可见" 是一个值得作为 default 复制的工程范式。它把共享降到"按需广播 + 隔离响应"两条窄通道，而非"全局共享 memory"。这与 thesis "Shared Workspace + BKN 共享 Memory 对 FC2 的工程级缓解" 目标一致。
- §3 footnote 2 给出了 β_r 私有的设计动机——"防止 sub-agent 之间相互影响"。这个动机在 thesis 关注的"协同失真"语境下是直接可引的工程论据。

**对 goal 4 (Composer) 的核心反直觉点提供初步支持**：thesis 把 Composer 明确限定为"基于已有 Agent/Template/Skill 的轻量组合"，明确不做端到端 spawn / DAG 引擎 / A2A 协议。Composer 的核心反直觉点是"主 agent 不必精确知道每个被组合 agent 的能力 detail"——[9] 在 4 backbone × 3 数据集上一致表明这一点工程上可行。**可借鉴的具体机制**：让 Composer 派生的 Plan 节点以"语义请求"形式发布，候选 Agent/Template 通过自身 BKN-anchored capability summary 自荐，由 Composer plan 节点仲裁选 1–N 个；这避免了 Composer 维护"全平台 Agent 能力索引"的负担。

**治理张力是必须正面回应的设计警告**：Blackboard 的 β_r 私有阻断 helper 响应进入跨 agent trace 视图，与 thesis "TraceAI-first / action provenance must be explicit" 不直接对齐。Decision Agent 落地 Shared Workspace 时需补设计——"β_r 私有 + 全量 trace 镜像到 TraceAI"，使治理审计独立于工程隔离；或采取分类型处理（只读响应走 β_r 私有 + trace 镜像；可写响应必须直接进入共享 β 并触发 Human-in-the-loop）。详见 `contradictions.md §Contradiction: Shared Workspace 私有响应板与 TraceAI 全审计的设计张力`。

**Blackboard 对 OneFlow [4] 的反向证据**：[9] 的 DS-GRU baseline（"把所有文件塞 prompt"，对应 OneFlow 的 folded MAS）在 Qwen3 上仅 1.21%、Claude 4 Opus 上仅 4.11%；Blackboard 在同 backbone 上分别取 7.90%、31.43%。这不是对 OneFlow 形式化等价的反驳，而是揭示 OneFlow 等价证明依赖"任务 fits 单 LLM 上下文"的隐含前提——当数据湖文件总量远超上下文窗口时，folded MAS 物理上无法承载等价构造。两者属边界条件互补：OneFlow 适用于上下文充足任务，Blackboard 适用于上下文必然溢出场景。这给 §0 异质性审计加了一条新维度——**"上下文 fit / 不 fit"也应是判据**。

**[9] 的关键缺陷限制其向企业规模的迁移性**：
- **"Helper agent autonomy" 实质是 prompt 二选一判断**，无分布式控制 / 投票 / 仲裁协议——是命名修辞而非架构属性 [9: §3.2 + Appendix C Figure 6]。
- **多 helper 响应冲突时主智能体仲裁机制完全不透明**，case study 显示 8 helper 中 3 个同时响应 [9: §3 + Appendix E]。
- **主实验默认聚类（Gemini-2.5-Pro 文件名）被自家消融实验证明劣于 E5-Large 内容嵌入 + KMeans**——主结果数字是 sub-optimal 配置下的，"+13–57%" 稳健性受影响。
- **Master–Slave baseline 是刻意削弱版本**（ReAct 主智能体 + 指名调用），未与 LangGraph supervisor / CrewAI hierarchical 等工业 orchestration 对比。
- **cluster 上限 27 / k≤16，远低于企业 5k–10k 规模**——broadcast 成本随 helper 数量线性放大，论文从未触及这一上界。
- **Cost 隐藏 offline 摊销**：每次数据湖更新需重做 file agent 离线分析，企业场景下不可忽略。
- **Search agent 仅查 Google CSE**，企业场景大量领域知识在内部 KB / 内部 wiki / 受控数据库中，模式无法直接平移。

**新增可证伪点**："Decision Agent Composer 的'无能力注册表'设计在企业 5k–10k Agent 规模仍成立"——本论文最大 helper agent 数为 27，证据支持仅适用于 ≤30 helper 的小规模。可证伪点表中 "Shared Workspace 能显著降低 FC2 协同失真" 状态从"未验证"更新为"间接工程证据，缺直接失败模式量化"——[9] 未引用 MAST [6]、未把 14 个 FM 作为评测靶点。

---

## 几个层次叠加 + 第零层先决条件 + 横切诊断维度：Decision Agent 工程问题地图

九篇论文叠加后，Decision Agent 系统面临的挑战可以按"调用生命周期 + 多智能体结构先决 + 通信拓扑 + 时序调度 + 横切诊断"组织：先回答第零层结构先决（多智能体是否必要），再依生命周期推进到调用前检索（§1）、智能体间路由（§2 / §2'）、智能体内技能/规划（§3/§4）；MAST [6] 横切到所有层，HSC [8] 在跨步时序维度构成反证案例。

| 层次 | 问题 | 现有答案 | 研究成熟度 |
|------|------|---------|-----------|
| **§0 结构先决** | 多智能体结构是否必要（同质 vs. 异质 / 上下文 fit vs. 溢出）？ | OneFlow 证明同质等价 [4]；Blackboard 反证上下文溢出场景 [9] | 单会话基准已验证；企业多会话未覆盖 |
| **§1 调用前检索** | 给定查询/子步，选哪些 agent/工具？ | Agent-as-a-Graph 类型化 KG + 加权 RRF [5]；MCP-Zero 模型主笔请求 [7] | 527 工具 + 2,797 needle 已验证；千级以上未覆盖；强模型 baseline 已饱和 |
| §2 智能体间路由（输出协调） | 选定的 worker 之间如何协同输出？ | GoA 的有向图消息传递 [2] | 单轮 QA 已验证；多步任务未覆盖 |
| **§2' 通信拓扑（请求分发）** | 主智能体如何把请求分发给 helper？ | Blackboard MAS 的 broadcast + 自荐 + β_r 隔离 [9] | ≤30 helper、单会话已验证；企业 5k–10k 未覆盖 |
| §3 智能体内技能 | Worker 知道做什么（动作抽象）？ | COS-PLAY 的技能库协同进化 [1] | 游戏域已验证；开放任务未覆盖 |
| §4 智能体内规划 | Worker 如何在动作间选择（规划深度）？ | FLARE 的前瞻 MCTS [3] | 图遍历/玩具环境已验证；开放任务未覆盖 |
| **§5 时序调度** | Agent 何时进入推理 / 反思 / 空闲态认知？ | HSC 概念框架 [8]（**反证案例：实验完全脱节**） | 学术稀缺已确认；机制有效性无证据 |
| **§⊥ 横切诊断** | 失败分布在哪里、多大、归因哪类？ | MAST 14 FM × 3 FC 分类法 [6] | 7 个开源 MAS、单会话已验证；闭源/企业未覆盖 |

**§0 与 §4 的交互**：[4] 和 [3] 合并提供了一个合成推断（两篇论文均未明确）：多智能体任务分解工作流可折叠为单智能体顺序推理而不损精度 [4]，而单智能体顺序推理本身在长时序任务中因步进式推理结构性不足而失败 [3]。因此，性能天花板由规划机制（§4）决定，而非多智能体通信拓扑（§2）。如果这一推断成立，Decision Agent 的首要工程投入应是 worker 内部规划质量，而非 worker 间协同协议 [low: 合成推断，无直接实验验证]。

**§1 与 §0 的张力**：[5] 在多智能体框架内默认其价值并优化路由层；[4] 质疑同质多智能体的必要性。若 [4] 同质等价结论在 Decision Agent 上成立，[5] 的检索方案失去前提（同质单智能体无需"选 agent"）；若异质性成立，[5] 的类型化召回直接服务于异质 worker 的预选。两者在 Decision Agent 的具体配置审计上互相绑定。

**§⊥ 与 §3 的呼应**：MAST [6] 的 FM-1.5 终止条件无感知（12.4%）和 FM-3.1 过早终止（6.2%）在执行层经验性地佐证了 FLARE [3] 的规划深度论断——这两类失败本质上是"规划者不知道何时停 / 在错的地方停"，是步进式推理在系统执行层留下的指纹 [6: Figure 1; 3: §3.2]。MAST 自己没把这层关系点破，但当 18.6% 的失败属于终止类问题时，前瞻规划机制就不是可选项。

**§⊥ 与 §0 的呼应**：MAST 度量到的 FC2 智能体间错位 32.3% 在 OneFlow [4] 的单 LLM 模拟器构造中被结构性消除——单智能体没有 FC2 的物理基础。这给"多智能体必要性审计"加了一条新维度：异质性不是唯一判据，FC2 失败率本身也应是判据。如果 Decision Agent 在自家轨迹上跑出 FC2 < 5%，那么"workers 协同"就不是核心架构挑战；如果 FC2 > 20%，论题前提即使在异质性满足的情况下也仍站得住。

**§2' 与 §⊥ 的对接**：Blackboard MAS [9] 在设计层面给出 master–slave 的替代方案（broadcast + 响应隔离 β_r），主张缓解 MAST 的 FM-2.1 信息隐瞒 / FM-2.4 信息共享错误 / FM-2.6 推理-动作错位——但 [9] 未引用 [6]、未把 14 个 FM 作为评测靶点，"Blackboard 缓解了哪些 FM / 引入了哪些新 FM" 是空白。Decision Agent 自家的 Shared Workspace 落地必须把 MAST 失败模式作为评测靶点，不能假设广播+隔离自动消解 FC2。

**共同空白**：九篇论文均未在同一系统中同时解决第零层、横切诊断与四层执行机制问题；均未在开放式真实任务（动态工具集、审批门、延迟奖励）中验证；均未测量异质多智能体协同在企业场景下的精度-成本曲线；MAST 提供了诊断范式但其结论分布（44/32/24）不可直接迁移到企业场景；Agent-as-a-Graph 验证了类型化召回但 527 工具规模远低于 ContextLoader 真正起作用的量级；Blackboard MAS 验证了广播-自荐通信但 cluster 上限 27 / helper ≤30，远低于企业 Composer 5k–10k 规模目标；HSC 给出了 Heartbeat 概念框架但实验完全脱钩——goal 2 的实证基础需要由 Decision Agent 团队自行建立。论题的核心部署主张——decision agent 作为独立组件与 openclaw 形成 manager-worker 关系——目前只有间接支持证据（COS-PLAY 技能库无状态运行 [1: §Relations]），尚无端到端的开放任务实证。

**对 Decision Agent 设计的具体落地建议（基于九篇论文合成）：**

1. **优先做一次自家版 MAST 度量**：在 BKN+ContextLoader+Dolphin+ISF 的真实业务轨迹上做 Grounded Theory 编码 + IAA 验证，得到 Decision Agent 自己的 FM 分布。Open-source MAS 的 44/32/24 是参考点，不是目标 [6: §3]。
2. **同时做异质性审计 + 上下文溢出场景识别**：审 BKN/ContextLoader/Dolphin/ISF 的基础模型与适配器，确认是否满足 OneFlow [4] 的"|B(W)| > 1"条件；同时识别企业长链任务中"上下文必然溢出"的子集（[9] 反证 OneFlow 折叠等价主张的边界条件）。三个审计结果（FC 分布 + 异质性 + 上下文溢出）合并起来才能判定多智能体架构是否仍是核心挑战 [4: §3.1; 9: §4.2 Table 1]。
3. **Dolphin 状态机的规划层升级**：FLARE [3] 的前瞻 MCTS 应作为 Dolphin 在长链 AutoFlow 中的规划骨架，而不仅仅是状态转移机。MAST 的 FM-1.5/FM-3.1 共 18.6% 是直接动机 [3: §5; 6: Figure 1]。
4. **ContextLoader 工具召回的双层验证 + 主笔请求改写前端**：(a) 在自家工具集复测 Agent-as-a-Graph 类型化结构 vs. 拼接基线，确认 +14.9% Recall@5 可迁移 [5: §4.2]；(b) 在 BKN 后端前置 MCP-Zero 风格请求改写器（"用户查询 → 主笔语义请求"），承担模糊查询的桥接 [7: §5.3]；(c) 用 MAST FM-2.4/FM-2.5/FM-2.6 三类失败模式作为定向预防目标 [6: §4 FC2]。
5. **Shared Workspace 的"β/β_r + TraceAI 镜像"设计**：以 Blackboard [9] 的"主智能体广播 + helper 自荐 + β_r 隔离" 作为 default 设计；但必须叠加"全量 trace 镜像到 TraceAI" 使治理审计独立于工程隔离；可写响应必须直接进入共享 β + Human-in-the-loop。该设计要求自带 MAST 失败模式追踪 [9: §3 footnote 2; thesis goal 3]。
6. **Composer 的"语义请求 + 候选 Agent 自荐"设计**：以 Blackboard [9] 的"主 agent 不必精确知道 sub-agent 能力 detail"作为 Composer 的核心机制依据，避免向通用 DAG 编排漂移；但本论文 helper 上限 27，企业 5k–10k 规模未验证——需要 Decision Agent 团队自行复现 scalability 主张 [9: §4.2; thesis goal 4]。
7. **Heartbeat + Cron 在治理边界内实现有限主动模式**：HSC [8] 揭示学术稀缺确实存在但实验完全脱钩，可吸收概念语言资源但不可作为机制有效性证据。Decision Agent 应限定 Reflection 在"建议生成"层而非"自主执行"层；主动行为必须有显式调度策略来源（cron / 用户授权 Reflection / Composer Plan 派生），不允许 intrinsic goal generation 进入生产路径 [8: §IV-C; thesis goal 2 修订]。

---

## 可证伪点追踪

论题声明若发现"主流 decision agent 必须依附于长期助理上下文才能闭环"，需修正。FLARE [3] 提供了第一个间接证据方向：若 worker 内部规划需要可靠的评估信号（r̂），而这个信号只有长期助理（openclaw）持有，那么 worker 对 manager 的依赖程度将远超"调度入口"，更接近"持续上下文依赖"。FLARE 本身未探讨这一场景，但其假设（规划时可用 r̂）恰好暴露了该风险点 [3: §Assumptions]。

OneFlow [4] 新增了论题的第二个可证伪方向：若 Decision Agent 的 workers 经审计属于同质配置（同一基础 LLM + 不同 prompt），则论题"多智能体 workers team 是核心架构挑战"将被部分证伪——单智能体工作流在精度上等价且成本更低。证伪路径：审计 BKN + ContextLoader + Dolphin 的基础模型是否共享。若共享：修正论题，将架构优先级从"多智能体协同机制"转向"单智能体工作流设计 + 前瞻规划"。若不共享：异质性得证，论题前提成立 [4: §3.1, Proposition 1]。

Agent-as-a-Graph [5] 为论题提供了一个**支持性**而非证伪性的可验证预测：BKN "逻辑/行动分离"语义解耦相对"拼接单向量"在召回精度上有显著优势。证伪路径：若在 KWeaver 自身工具集上复测，类型化结构 vs. 拼接基线的 Recall@K 差距小于 5%（明显低于 [5] 报告的 14.9%），则 BKN 的实证支撑被削弱，需要寻找其他防御机制。若差距与 [5] 一致或更大：BKN 设计获得跨数据集的可重复证据 [5: §4.2]。

MAST [6] 新增了论题的第三个可证伪方向，与 OneFlow 形成"双判据"：在 Decision Agent 自家轨迹上跑 MAST 范式（Grounded Theory + IAA + LLM-as-Judge），度量 FC2 智能体间错位的实际占比。若 FC2 ≪ MAST 报告的 32.3%（例如 < 5%），即使 §0 异质性审计通过，"协同失真是核心挑战"这一论题成分也被部分证伪——失真已被 BKN+Dolphin 工程化解决，剩余瓶颈在别处（更可能是 FC1 系统设计或 FC3 校验）。若 FC2 处于或高于 32.3%，论题前提保留，并应将 14 个 FM 作为 Decision Agent 可观测性 SLA 的对标基线 [6: §4, Figure 1]。

附带：MAST 也具体化了一条早被论题提及但未明确的可证伪点——thesis §1 "缺乏学术层面的系统性验证"措辞本身。MAST 的 1642 轨迹 + κ=0.88 + κ=0.79 域外验证已构成对协同失真维度的系统性验证；论题该处措辞需更新为"通用 MAS 已有基线，企业场景待对标"。具体处理选项见 `contradictions.md §Contradiction: 协同失真验证缺口已被部分填补`，由人类决定 Option A/B/C 哪个落地。

MCP-Zero [7] 修正了一条隐含的过度承诺——"主动检索能提升精度"。证伪路径已实现：GPT-4.1 等强模型上 baseline 已饱和，MCP-Zero 仅省 token 无精度增益。论题已据此修订（thesis §"ContextLoader 的动态按需加载机制" 段加入"强模型档位的实际增益更可能落在 token 成本侧而非精度侧"边界）。

HSC [8] 暴露了 thesis goal 2 与治理优先核心定位的结构性张力——HSC Dream Mode 的"自主目标设定 / 合成经验生成"违反 ISF/TraceAI 的 "action provenance must be explicit"。论题已据此修订（goal 2 重构为"在治理边界内实现有限主动模式"），并把"学术稀缺 = 机会窗口"默认假设降级为"稀缺需要解释，不默认机会信号"。新可证伪点："Goal 2 客户真实需求是'主动认知活动'而非'定时任务调度'"——若 ≥50% "7×24 数字员工"客户访谈实际只需要 cron + 失败重试，goal 2 应从增强能力降级为运维能力。

Blackboard MAS [9] 对论题的影响有三条：(1) 给 thesis 可证伪点 "Shared Workspace 能显著降低 FC2 协同失真" 提供间接工程证据（设计层缓解 FC2，但 [9] 未量化失败模式），状态从"未验证"更新为"间接工程证据，缺直接失败模式量化"；(2) 在 OneFlow [4] 的"上下文必然溢出"边界条件上反证单 LLM 折叠等价主张——证伪路径：识别 Decision Agent 企业长链任务中的上下文溢出子集，验证 Blackboard 风格 broadcast 通信在这类子集上是否仍优于单智能体顺序执行；(3) 新增可证伪点"Decision Agent Composer 的'无能力注册表'设计在企业 5k–10k Agent 规模仍成立"——本论文 helper 上限 27，证据仅适用 ≤30 helper 小规模，超出该规模的 broadcast 成本与仲裁稳定性需 Decision Agent 团队自行复测。详见 `contradictions.md §Contradiction: Shared Workspace 私有响应板与 TraceAI 全审计的设计张力` 与 `§Contradiction: 多智能体折叠等价（OneFlow）vs 上下文必然溢出场景的反例`。

---

## 版本更新日志

| 版本 | 日期 | 新增论文 | 关键变化 |
|------|------|---------|---------|
| v1 | 2026-04-27 | [01] COS-PLAY | 建立技能库协同进化基线；独立部署初步支持 |
| v2 | 2026-04-27 | [02] GoA | 增加推理时图路由参考；调度链轻量实现讨论 |
| v3 | 2026-04-27 | [03] FLARE | 增加单智能体规划机制层；步进式推理结构性局限诊断；可证伪点首次具体化 |
| v4 | 2026-04-27 | [04] OneFlow | 新增多智能体必要性挑战；引入§0结构先决层；可证伪点二（同质配置审计）具体化 |
| v5 | 2026-04-29 | [05] Agent-as-a-Graph | 新增调用前检索层；BKN 语义解耦的首份学术实证（+14.9% Recall@5）；规模缺口（527 工具）作为 Decision Agent 团队可填补的研究空白 |
| v6 | 2026-04-30 | [06] MAST | 新增§⊥横切诊断维度；协同失真从"未度量"→"已有通用基线"；可证伪点三（自家 FC2 度量）具体化；Decision Agent 工程落地 4 条建议 |
| v7 | 2026-04-30 | [07] MCP-Zero | 主动工具检索的边界条件——强模型 baseline 已饱和；BKN 语义解耦第二条独立证据线；修正"主动检索能提升精度"过度承诺；建议工程化组合 [5] 后端 + [7] 前端 |
| v8 | 2026-05-01 | [08] HSC | 新增§5 时序调度层（反证案例）；揭示 goal 2 与治理优先的结构性张力；thesis 修订 goal 2 为"治理边界内有限主动模式"；学术稀缺信号降级为"需要解释，不默认机会信号" |
| v9 | 2026-05-02 | [09] Blackboard MAS | 新增§2' 通信拓扑层；goal 3 (Shared Workspace) 与 goal 4 (Composer) 的首份工程化原型；β_r 私有 vs TraceAI-first 设计张力；OneFlow 边界条件反证（上下文必然溢出场景）；Decision Agent 工程落地建议从 4 条扩至 7 条 |
