# Decision Agent: Research Report

> **Version:** v12 (12 papers)
> **Last Updated:** 2026-05-02
> **Papers:** [01](notes/01_co_evolving_llm_decision_and_skill.md), [02](notes/02_graph_of_agents_a_graph_based.md), [03](notes/03_why_reasoning_fails_to_plan_a.md), [04](notes/04_rethinking_the_value_of_multi_agent.md), [05](notes/05_agent_as_a_graph_knowledge_graph.md), [06](notes/06_why_do_multi_agent_llm_systems.md), [07](notes/07_mcp_zero_active_tool_discovery_for.md), [08](notes/08_simulating_human_cognition_heartbeat_driven_autonomous.md), [09](notes/09_llm_based_multi_agent_blackboard_system.md), [10](notes/10_collaborative_memory_multi_user_memory_sharing.md), [11](notes/11_retrieval_models_aren_t_tool_savvy.md), [12](notes/12_automated_composition_of_agents_a_knapsack.md)
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

### 第二条独立证据：MCP-Zero 的"模型主笔结构化请求"

[7] 从相反方向给 BKN 语义解耦提供第二条证据线。范式是让 LLM 在执行中输出 `<tool_assistant>{server, tool}` 结构化请求触发外部检索，再用"server→tool"两段式 Hierarchical Vector Routing 召回。APIBank 多轮 Full 集（48 unique tools）上"模型主笔请求 → top-1 工具" 90.32%，相对"用户查询直接检索"基线高 25 个百分点（90.32% vs. 65.05%）；token 消耗从 6402.2 降至 159.0（−97.52%）[7: §5.3, Table 1]。在 1–2,797 工具大海捞针实验中，弱模型档位（Claude-3.5-Sonnet、Gemini-2.5-Flash）显著超越 schema 全注入 baseline [7: §5.2, Figure 5]。

**与 [5] 的对照与互补**。两者都把"语义解耦 / 显式分离"作为对抗工具幻觉的核心机制——[5] 用类型化 KG 做被动召回结构，[7] 用 LLM 主笔生成对齐请求做主动召回输入；都不是更换 embedding 而是改变信号源结构。但 LiveMCPBench 上 [5] Recall@5=0.85 系统性击败 [7] 0.70 [5: §4.2 Table 1]，说明"被动结构化召回"在精度上仍优于"主动检索请求"——MCP-Zero 的真正杠杆在 token 成本（弱模型 −97.52%）与弱模型精度补偿（强模型已饱和无增益）。

**[7] 揭示的两条边界条件**对论题至关重要：
1. **强模型档位精度饱和**。GPT-4.1 在 schema 全注入 baseline 下已能达 91.40–92.47% 精度，MCP-Zero 在该档位"无显著精度增益，仅省 token" [7: §5.2 Figure 5; §5.3 Table 1]。这意味着 BKN/ContextLoader 的精度防御杠杆主要在弱模型与高度受限的成本目标（token 预算 < 1k/会话），强模型 + 充足上下文场景的真正瓶颈在别处（更可能是 §3 规划深度而非召回精度）。
2. **APIBank 48 工具的规模差距**。论文报告的 90.32% 在 48 工具上得到，与 [5] 的 527 工具与 ContextLoader 目标的 5k–10k 之间各差一个数量级，"主动检索精度增益跨规模递减"的可能性未被排除 [7: §Weaknesses]。

**对 Decision Agent 的工程含义**：(a) ContextLoader 应将"BKN 类型化召回（[5] 路径）"作为默认结构，"模型主笔结构化请求（[7] 路径）"作为弱模型场景或高 token 压力场景的开关而非默认；(b) 强模型 + 长上下文档位的真实成本-精度曲线需要自家测，不可继承 [7] 的弱模型结论；(c) thesis "工具幻觉是反直觉难题"措辞需要精细化——它在弱模型档位是真难题，在强模型档位是 token 预算难题。

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

## 几个层次叠加 + 第零层先决条件 + 横切诊断维度：Decision Agent 工程问题地图

六篇论文叠加后，Decision Agent 系统面临的挑战可以按"调用生命周期 + 多智能体结构先决 + 横切诊断"组织：先回答第零层结构先决（多智能体是否必要），再依生命周期推进到调用前检索（§1）、智能体间路由（§2）、智能体内技能/规划（§3/§4）；MAST [6] 横切到所有层，提供失败诊断的可观测性维度。

| 层次 | 问题 | 现有答案 | 研究成熟度 |
|------|------|---------|-----------|
| **§0 结构先决** | 多智能体结构是否必要（同质 vs. 异质）？ | OneFlow 证明同质等价 [4] | 单会话基准已验证；企业多会话未覆盖 |
| **§1 调用前检索** | 给定查询/子步，选哪些 agent/工具？ | Agent-as-a-Graph 类型化 KG + 加权 RRF [5] | 527 工具量级已验证；千级以上未覆盖 |
| §2 智能体间路由 | 选定的 worker 之间如何协同输出？ | GoA 的有向图消息传递 [2] | 单轮 QA 已验证；多步任务未覆盖 |
| §3 智能体内技能 | Worker 知道做什么（动作抽象）？ | COS-PLAY 的技能库协同进化 [1] | 游戏域已验证；开放任务未覆盖 |
| §4 智能体内规划 | Worker 如何在动作间选择（规划深度）？ | FLARE 的前瞻 MCTS [3] | 图遍历/玩具环境已验证；开放任务未覆盖 |
| **§⊥ 横切诊断** | 失败分布在哪里、多大、归因哪类？ | MAST 14 FM × 3 FC 分类法 [6] | 7 个开源 MAS、单会话已验证；闭源/企业未覆盖 |

**§0 与 §4 的交互**：[4] 和 [3] 合并提供了一个合成推断（两篇论文均未明确）：多智能体任务分解工作流可折叠为单智能体顺序推理而不损精度 [4]，而单智能体顺序推理本身在长时序任务中因步进式推理结构性不足而失败 [3]。因此，性能天花板由规划机制（§4）决定，而非多智能体通信拓扑（§2）。如果这一推断成立，Decision Agent 的首要工程投入应是 worker 内部规划质量，而非 worker 间协同协议 [low: 合成推断，无直接实验验证]。

**§1 与 §0 的张力**：[5] 在多智能体框架内默认其价值并优化路由层；[4] 质疑同质多智能体的必要性。若 [4] 同质等价结论在 Decision Agent 上成立，[5] 的检索方案失去前提（同质单智能体无需"选 agent"）；若异质性成立，[5] 的类型化召回直接服务于异质 worker 的预选。两者在 Decision Agent 的具体配置审计上互相绑定。

**§⊥ 与 §3 的呼应**：MAST [6] 的 FM-1.5 终止条件无感知（12.4%）和 FM-3.1 过早终止（6.2%）在执行层经验性地佐证了 FLARE [3] 的规划深度论断——这两类失败本质上是"规划者不知道何时停 / 在错的地方停"，是步进式推理在系统执行层留下的指纹 [6: Figure 1; 3: §3.2]。MAST 自己没把这层关系点破，但当 18.6% 的失败属于终止类问题时，前瞻规划机制就不是可选项。

**§⊥ 与 §0 的呼应**：MAST 度量到的 FC2 智能体间错位 32.3% 在 OneFlow [4] 的单 LLM 模拟器构造中被结构性消除——单智能体没有 FC2 的物理基础。这给"多智能体必要性审计"加了一条新维度：异质性不是唯一判据，FC2 失败率本身也应是判据。如果 Decision Agent 在自家轨迹上跑出 FC2 < 5%，那么"workers 协同"就不是核心架构挑战；如果 FC2 > 20%，论题前提即使在异质性满足的情况下也仍站得住。

**共同空白**：六篇论文均未在同一系统中同时解决第零层、横切诊断与四层执行机制问题；均未在开放式真实任务（动态工具集、审批门、延迟奖励）中验证；均未测量异质多智能体协同在企业场景下的精度-成本曲线；MAST 提供了诊断范式但其结论分布（44/32/24）不可直接迁移到企业场景；Agent-as-a-Graph 验证了类型化召回但 527 工具规模远低于 ContextLoader 真正起作用的量级。论题的核心部署主张——decision agent 作为独立组件与 openclaw 形成 manager-worker 关系——目前只有间接支持证据（COS-PLAY 技能库无状态运行 [1: §Relations]），尚无端到端的开放任务实证。

**对 Decision Agent 设计的具体落地建议（基于六篇论文合成）：**

1. **优先做一次自家版 MAST 度量**：在 BKN+ContextLoader+Dolphin+ISF 的真实业务轨迹上做 Grounded Theory 编码 + IAA 验证，得到 Decision Agent 自己的 FM 分布。Open-source MAS 的 44/32/24 是参考点，不是目标 [6: §3]。
2. **同时做异质性审计**：审 BKN/ContextLoader/Dolphin/ISF 的基础模型与适配器，确认是否满足 OneFlow [4] 的"|B(W)| > 1"条件。两个审计结果（FC 分布 + 异质性）合并起来才能判定多智能体架构是否仍是核心挑战 [4: §3.1]。
3. **Dolphin 状态机的规划层升级**：FLARE [3] 的前瞻 MCTS 应作为 Dolphin 在长链 AutoFlow 中的规划骨架，而不仅仅是状态转移机。MAST 的 FM-1.5/FM-3.1 共 18.6% 是直接动机 [3: §5; 6: Figure 1]。
4. **ContextLoader 工具召回的双层验证**：(a) 在自家工具集复测 Agent-as-a-Graph 类型化结构 vs. 拼接基线，确认 +14.9% Recall@5 可迁移 [5: §4.2]；(b) 用 MAST FM-2.4/FM-2.5/FM-2.6 三类失败模式作为定向预防目标，BKN 语义结构化解耦的有效性需在自家度量中验证 [6: §4 FC2; references/02_decision_agent_tool_management.md]。

---

## 可证伪点追踪

论题声明若发现"主流 decision agent 必须依附于长期助理上下文才能闭环"，需修正。FLARE [3] 提供了第一个间接证据方向：若 worker 内部规划需要可靠的评估信号（r̂），而这个信号只有长期助理（openclaw）持有，那么 worker 对 manager 的依赖程度将远超"调度入口"，更接近"持续上下文依赖"。FLARE 本身未探讨这一场景，但其假设（规划时可用 r̂）恰好暴露了该风险点 [3: §Assumptions]。

OneFlow [4] 新增了论题的第二个可证伪方向：若 Decision Agent 的 workers 经审计属于同质配置（同一基础 LLM + 不同 prompt），则论题"多智能体 workers team 是核心架构挑战"将被部分证伪——单智能体工作流在精度上等价且成本更低。证伪路径：审计 BKN + ContextLoader + Dolphin 的基础模型是否共享。若共享：修正论题，将架构优先级从"多智能体协同机制"转向"单智能体工作流设计 + 前瞻规划"。若不共享：异质性得证，论题前提成立 [4: §3.1, Proposition 1]。

Agent-as-a-Graph [5] 为论题提供了一个**支持性**而非证伪性的可验证预测：BKN "逻辑/行动分离"语义解耦相对"拼接单向量"在召回精度上有显著优势。证伪路径：若在 KWeaver 自身工具集上复测，类型化结构 vs. 拼接基线的 Recall@K 差距小于 5%（明显低于 [5] 报告的 14.9%），则 BKN 的实证支撑被削弱，需要寻找其他防御机制。若差距与 [5] 一致或更大：BKN 设计获得跨数据集的可重复证据 [5: §4.2]。

MAST [6] 新增了论题的第三个可证伪方向，与 OneFlow 形成"双判据"：在 Decision Agent 自家轨迹上跑 MAST 范式（Grounded Theory + IAA + LLM-as-Judge），度量 FC2 智能体间错位的实际占比。若 FC2 ≪ MAST 报告的 32.3%（例如 < 5%），即使 §0 异质性审计通过，"协同失真是核心挑战"这一论题成分也被部分证伪——失真已被 BKN+Dolphin 工程化解决，剩余瓶颈在别处（更可能是 FC1 系统设计或 FC3 校验）。若 FC2 处于或高于 32.3%，论题前提保留，并应将 14 个 FM 作为 Decision Agent 可观测性 SLA 的对标基线 [6: §4, Figure 1]。

附带：MAST 也具体化了一条早被论题提及但未明确的可证伪点——thesis §1 "缺乏学术层面的系统性验证"措辞本身。MAST 的 1642 轨迹 + κ=0.88 + κ=0.79 域外验证已构成对协同失真维度的系统性验证；论题该处措辞需更新为"通用 MAS 已有基线，企业场景待对标"。具体处理选项见 `contradictions.md §Contradiction: 协同失真验证缺口已被部分填补`，由人类决定 Option A/B/C 哪个落地。

---

## Design Goal 视角整合（v7–v12 backfill）

> 本节按 thesis Design Context 的 5 个 design goals 重新组织 [8]–[12] 的影响。早先章节按"调用生命周期 / 第零层先决 / 横切诊断"的学术轴组织，这里按 Decision Agent 真实工程决策面组织——便于在落地评审时直接对应到 0.8 路线图的能力槽位。

### Goal 2 — Heartbeat + Cron 主动 Runtime：HSC 是反证案例 [8]

[8] (HSC) 是迄今唯一以 "heartbeat scheduling" 直接命名的 LLM agent 工作。其 §III–IV 构想 LLM agent + 可插拔认知模块（Planner/Critic/Recaller/Dreamer）+ Dream Mode 三件事（记忆压缩 / 合成经验 / 自主目标设定）+ 双层调度（macro/micro）+ 状态增强 `s_t' = φ(s_t, h_t)` 防重复循环，但 §VI 实验完全脱节——LSTM 在合成 1,800 天 × 24h × 6 类动作序列上做监督 next-action 预测，无 LLM、无认知模块实例化、无 Dream 触发、无任何 baseline 对比 [8: §III-IV vs §VI]。

**对论题影响**：
- **学术稀缺信号被部分证伪**：thesis 原 Taste 假设"主动 Agent 学术稀缺=机会窗口"——[8] 揭示更可能的解释是"问题在企业场景下可由 cron + ReAct + 治理围栏简单组合解决"（同 OneFlow 对多 agent 必要性的证伪逻辑）。Goal 2 的工程优先级因此应从"补足空白"调整为"在治理边界内实现有限主动模式"。
- **治理张力被首次具体化**：HSC 的 Dream Mode 自主目标设定违反 ISF/TraceAI "action provenance must be explicit" 治理约束——主动 agent 越主动，治理越被动。thesis 已据此修订（governance-aware bounded autonomy），保留 Reflection 类能力但限定在"建议生成"层而非"自主执行"层。
- **可吸收的语言资源（不可吸收实验）**：HSC 概念命名（macro/micro 双层调度、Dream Mode 三大功能、复合奖励 `R_total = α·R_external + β·R_internal`）、状态增强 `s_t' = φ(s_t, h_t)` 防 MAST FM-1.3 步骤重复的形式化参考——可作为 Dolphin 状态机引入近 k 步动作哈希的语言起点，但不可被引用为"主动调度有效"的证据 [8: §V-C]。

### Goal 3 + Goal 4 — Shared Workspace 与 Composer：Blackboard 提供候选原型，Knapsack 提供 design-time 验证 [9][12]

**[9] Blackboard MAS**：β / β_r 双板分离（共享黑板 β 接受请求广播，私有响应板 β_r 隔离 helper 响应防 cross-influence）在 master-slave 基线上获 13–57% 相对提升（KramaBench / DSBench / DA-Code × 4 backbone）；File discovery F1 0.561 vs Master–Slave 0.513 vs RAG 0.247 [9: §4.2 Tables 1–2]。Decision Agent 落地决议（thesis 已更新）：内部信息处理（检索 / 汇总 / 推理中间步）走 β_r 私有 + TraceAI 镜像，涉及外部副作用（写库 / 调用工具 / 发邮件）必须直接进 β + Human-in-the-loop。

**[9] 揭示的三个工程边界条件**：
1. **β_r 私有 vs TraceAI 全审计的张力**——需"β_r 私有 + 全量 trace 镜像到 TraceAI"双轨设计，使治理审计独立于工程隔离。
2. **"Helper autonomy" 是 prompt 修辞而非架构属性**——[9] helper LLM 被 prompt 二选一判断"你能否处理"，本质是 LLM-as-router；不能因此放弃 BKN-anchored capability lookup，需对照"广播自荐 vs 结构化 BKN 查询"在 5k+ 工具/Agent 规模下的 Recall@5。
3. **规模差两个数量级**——[9] helper 上限 27，Decision Agent 目标 5k–10k；broadcast 成本线性增长，5k 下不可行，需保留 BKN 预过滤回退路径。

**[9] 同时反证 OneFlow 折叠等价的边界条件**：DS-GRU baseline（Qwen3 1.21% / Claude 4 Opus 4.11%）在数据湖文件总量超出上下文窗口时反证 [4] 折叠等价物理上无法承载——OneFlow 适用于上下文充足任务，上下文必然溢出场景需多 agent 范式。详见 §0 多智能体结构是否必要章节。

**[12] Component Composition (Knapsack)** 给 goal 4 Composer 提供 design-time pipeline 原型：composer agent 工作流 = "把任务描述解析成 ≤6 个 skill → top-K=10 语义检索候选 → 沙箱实测（CodeAct/ReAct + composer LLM-as-judge）→ ZCL 在线阈值 Ψ(z) = (U / (L·e))^z · L·e 收/弃"，理论 ln(U/L)+1 竞争比 [12: §4, Algorithm 1]。单 agent Pareto 主结果（Claude 3.5 Sonnet）GAIA online knapsack+OPT($30) 0.47 success @ $14 vs identity 0.47 @ $398；多 agent 在 117 sub-agent inventory 上 mortgage overall 0.07→0.87 [12: §5, Figures 3–4]。

**[12] 的工程含义与新增可证伪点**：
- **Composer 三段式 pipeline 直接同构 thesis goal 4**：[12] 的"生成 skill → 语义检索候选 → 沙箱实测 → 预算约束选择" 与 thesis "自然语言描述任务 → 可审查 Agent Team Plan → 调度执行" 在结构上同构；Appendix A.1–A.3 prompt 模板可作为 Composer Plan 节点工程起点。
- **Sandbox-validated capability registry vs runtime self-nomination 的治理分轨**：[12] design-time 沙箱 vs [9] runtime self-nomination 是"capability registry 何时校准"的两种解法——前者更契合 thesis "action provenance must be explicit" 治理要求（沙箱轨迹可全量进 TraceAI），后者更契合 5k–10k 规模动态调度。Decision Agent 落地 Composer 时应取分轨：BKN 静态登记 + 周期性沙箱（[12] 路径）+ 运行时 last-mile 路由（[9] 路径）。
- **预算约束是多租户场景的天然拓扑**：knapsack 框架把约束从隐式 (token 预算) 提升为显式 (component cost) 决策变量，与 ContextLoader "工具规模 ×10 + 上下文 −90%" 工程指标可直接对接；但 [12] 多 agent 实验全 cost = $1（ZCL 退化为 value 阈值），自家 BKN 工具集 cost 跨 free→paid→infra 多档时 ZCL 竞争比 ln(U/L)+1 实际表现需复测。
- **新增可证伪点（建议加入 thesis 表）**：(1) "Composer 的 design-time sandbox validation 在 5k+ agent / 工具规模下沙箱总耗时可接受"——[12] 自报 10–30 min @ 117 agent，5k 规模线性外推 8–22 h，需检查能否分布式并行；(2) "Knapsack 公式中假设的'组件值可加'在 Decision Agent 多步 BKN AutoFlow 中不引入 >10% 选择错误"——[12] 自承非加性交互被 Algorithm 1 忽略。
- **GAIA 持平观察反向支持 ContextLoader 价值命题 [low]**：identity 122 工具 0.47 @ $398 与 online knapsack 4 工具 0.47 @ $14 的持平更可能由"122 工具撑爆 prompt 上下文"驱动（[12] 未做控制实验排除），这反向支持 ContextLoader "动态按需加载"的工程定位。

**[12] 与 [4] OneFlow 的第二条边界**：[12] §5.2 在 117 sub-agent + supervisor 设置下 identity 0.07 vs online knapsack 0.87，与 [9] DS-GRU 0.07-级 baseline 形成"OneFlow 折叠等价的两条边界条件"——[9] 上下文必然溢出场景物理无法承载、[12] supervisor delegation capacity 在 100+ 跨域 agent 场景被绑死。处理选项见 `contradictions.md §Contradiction: OneFlow 折叠等价的第二条边界`。

### Goal 5 — 跨会话层级化 Memory：Collaborative Memory 提供形式化原型 [10]

[10] 是当前 corpus 中**第一份**直接处理"多用户 memory 共享 + 时变访问控制"的工作。把单用户 monolithic memory 替换为带访问控制的双层 memory（M_private 按贡献用户隔离 + M_shared 按贡献 agent 分组）；每条 fragment 携带 provenance 四元组 (T, U, A, R)；时变授权由两张 bipartite graph G_UA(t) ⊆ U×A、G_AR(t) ⊆ A×R 刻画；可读 fragment 集合 M(u,a,t) = {m | A(m) ⊆ A(u,t) ∧ R(m) ⊆ R(a,t)} [10: §3.1, §3.2 Eq. 3]。

**对 Decision Agent 的可操作影响**：
- **三层记忆原型 [med]**：thesis 需 User / Role / Org 三层，[10] 给 User / Shared 两层缺 Role；但 M_shared(a, t) 按贡献 agent 分组实质是 Role 维度雏形——可类比为"按贡献 agent 集合切分的 shared memory 子区间"，再加 Org 层作为第三层。
- **写入双通道直接同构 [high]**：[10] π^{write/private} 与 π^{write/shared} 双 policy 与 thesis "private 直写 + shared 经审 / 转写"完全对齐；Table 1 两条系统级 prompt（private 保留具体细节、shared 强制移除用户特定信息）可作为双通道 prompt 设计的直接借鉴起点。
- **provenance ↔ TraceAI 对齐 [high]**：(T, U, A, R) 四元组与 thesis "TraceAI-first" 结构同构；但 [10] 实现层 provenance 是普通 metadata 无防篡改，Decision Agent 借鉴时**必须升级为 append-only + 哈希链 / 签名**才满足"事后审计可信"治理要求。
- **ISF 权限模型形式化 [high]**：G_UA(t) 与 G_AR(t) 双 bipartite graph 是 ISF 权限模型的具体数学形式，Eq. 3 的 M(u,a,t) 求交规则给出"在两层授权下何条 memory 可见"的可计算判据。
- **关键工程空缺需自行补齐**：thesis 治理与遗忘机制 / 跨用户冲突合并 / Role 层细分 / provenance 防篡改——[10] 全部缺失。
- **新增可证伪点（建议加入 thesis 表）**：(1) "User/Role/Org 三层记忆设计在 5k+ 工具 / 数百用户角色规模下，Eq. 3 求交计算保持亚线性"——[10] 最大配置 5×6×6，证据仅适用数十节点；(2) "LLM-based write policy（redact / anonymize）在企业敏感数据场景下失败率 ≤ 1%"——[10] 未做该评测，>5% 泄漏率即整个 transformation write 路径不可接受、需替换为规则引擎。

证据强度：[10] 全文 [med]——方法论清晰、形式化干净、3 场景递进合理；但实验规模小、provenance 无防篡改、安全保证依赖 LLM 概率、跨用户冲突未处理、无衰减、关键 ablation 缺失——**应作为 goal 5 access control / provenance 形式化原型来源，不应作为 LLM-based redaction 在企业治理场景下安全性的直接证据**。

### 工具检索专用微调：BKN + 通用 embedding 的精度上限 [11]

ToolRet [11] 跨 30+ 公开 tool-use 数据集汇总 7.6k 任务 / 43k 工具，按 BEIR/MTEB 风格统一格式，量化"现成 IR 模型在工具检索上系统性崩盘"：6.4B 参数级 NV-Embed-v1 平均 nDCG@10 仅 33.83 / Completeness@10 ≤ 35%，加 instruction 后提升至 42.71 / 43.41（+7~17 nDCG 点）；最强 cross-encoder bge-reranker-v2-gemma w/ inst. 47.52 / 48.90 仍意味 ≥51% multi-target query 无法在 top-10 全召回 [11: §6.1, Tables 4–5]。端到端 ToolBench-G1 把官方人工标注 toolset 换成 IR 召回结果后 GPT-3.5 pass rate 62.0→50.6（−11.4）；ToolRet-train（200k+ 指令式 + K=10 负采样）微调后 GPT-3.5 / ToolLlama pass rate 同步抬升 10–20 点 [11: §1, §7]。

**[11] 给 BKN/ContextLoader 的四点工程含义**：
1. **现成 dense 检索器不足以支撑 ContextLoader 5k–10k 规模**——NV-Embed-v1 在 43k 工具池仍只 33.83 nDCG@10；ToolRet-train 这类"指令式 + 多源 + K=10 负采样"工具检索专用微调是工程必需而非 nice-to-have。
2. **Instruction-augmented 协议 ↔ BKN 解耦 / [7] 主动请求同源**——[11] instruction = "面向 target tool 的功能描述 + 任务约束"，[7] `<tool_assistant>` 块 = "主动权限域 + 操作类型"；两条独立证据线（[11] +7~17 nDCG 跨 30+ 数据集 / [7] +25 pp APIBank Full）共同支持 thesis "仅靠用户原始 query 做工具检索是错误的设计点"。
3. **Completeness@10 ≤ 50% 是 ContextLoader 的设计警钟**——multi-target query 即使最强 reranker 也只 48.90% 全召回；ContextLoader 必须支持 K > 10 + 二段筛选（语义 retrieval → 结构化 BKN 过滤），而非单段 top-10。
4. **MTEB Pearson 0.79 + 绝对值低 ~20 点是可证伪点的具体设计**——若 KWeaver BKN 工具集复测中相关性 ≈ 0.79 + 绝对值 ≈ 低 20 点关系不再成立（如 BKN 解耦后绝对值反而提升），则可作为 BKN "语义结构化是工具检索专属归纳偏置"的工程级证据。

**新增可证伪点（建议加入 thesis 表）**："ContextLoader 仅做 BKN 类型化 + 通用 embedding（无工具检索专用微调）已足以支撑 5k+ 工具规模"——证伪条件：BGE/E5 等通用 embedding 在 BKN 增强后 Recall@10 仍 < 50%；当前状态：[11] 间接支持该证伪条件（通用 embedding 在 43k 工具池 Recall@10 ≤ 52%，且无 BKN 类型化辅助）。

### 工程问题地图（v12 更新）

下表是早先 §"几个层次叠加"工程地图的更新版，覆盖 12 篇论文。新增列"对应 design goal"建立学术轴 ↔ Decision Agent 0.8 能力槽位的映射。

| 层次 | 现有答案 | 对应 design goal | 研究成熟度 |
|------|---------|-----------------|-----------|
| §0 结构先决 | OneFlow 同质等价 [4]，但 [9] 上下文溢出 + [12] supervisor delegation 是两条边界 | — | 单会话基准已验证；企业多会话未覆盖 |
| §1 调用前检索（架构） | Agent-as-a-Graph 类型化 KG + 加权 RRF [5] | goal 3 (BKN 同构) | 527 工具量级；千级以上未覆盖 |
| §1 调用前检索（运行时主笔） | MCP-Zero 主笔结构化请求 [7] | goal 3 (ContextLoader) | 弱模型档位；强模型饱和无精度增益 |
| §1 调用前检索（评测/训练） | ToolRet IR benchmarking + ToolRet-train 微调 [11] | goal 3 (ContextLoader) | 43k 工具评测；ToolBench 端到端 |
| §1 调用前检索（design-time 验证） | Knapsack composer + ZCL + sandbox [12] | goals 1, 4 | 120 工具 / 117 agent；5k+ 未触及 |
| §2 智能体间路由 | GoA 有向图消息传递 [2] | goal 4 (Composer) | 单轮 QA 已验证；多步任务未覆盖 |
| §2 智能体间通信结构 | Blackboard β / β_r 双板 [9] | goals 3, 4 | helper 上限 27；规模 ≤ 30 |
| §3 智能体内技能 | COS-PLAY 技能库协同进化 [1] | goal 5 (Memory) | 游戏域；开放任务未覆盖 |
| §4 智能体内规划 | FLARE 前瞻 MCTS [3] | goals 1, 4 | 图遍历/玩具环境；开放任务未覆盖 |
| §5 跨会话记忆 + 权限 | Collaborative Memory bipartite graph [10] | goals 3, 5 | 5×6×6 节点规模；冲突 / 衰减缺失 |
| §6 时序调度（主动 Runtime） | HSC 心跳调度（**反证案例**）[8] | goal 2 | 实验与理论脱节；无 baseline |
| §⊥ 横切诊断 | MAST 14 FM × 3 FC 分类法 [6] | 跨所有 goals | 7 个开源 MAS、单会话；闭源/企业未覆盖 |

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
| v7 | 2026-05-01 | [07] MCP-Zero | 新增"调用前检索（运行时主笔）"路径；BKN 语义解耦第二条独立证据线；揭示强模型档位精度饱和的边界条件 |
| v8 | 2026-05-01 | [08] HSC | Goal 2 主动 Runtime 学术对照——但作为反证案例：单作者 / 实验与理论脱节 / 无 baseline；学术稀缺 ≠ 机会窗口被部分证伪；治理张力（Dream Mode 自主目标设定违反 action provenance）首次具体化 |
| v9 | 2026-05-01 | [09] Blackboard MAS | Goal 3 (Shared Workspace) + Goal 4 (Composer) 候选 default 设计 (β/β_r 双板)；OneFlow 折叠等价第一条边界（上下文必然溢出场景）；新增"内部信息处理 vs 外部副作用"分轨决议 |
| v10 | 2026-05-02 | [10] Collaborative Memory | Goal 5 跨会话层级化 Memory 第一份形式化原型；写入双通道 / provenance ↔ TraceAI / ISF bipartite graph 三处 [high] 同构；新增可证伪点 (Eq. 3 求交亚线性 / LLM-based redaction 失败率) |
| v11 | 2026-05-02 | [11] ToolRet | Goal 3 (ContextLoader) 工具检索专用微调必要性首次量化（NV-Embed-v1 nDCG@10 33.83→42.71 + ToolRet-train +10–20 pp）；Completeness@10 ≤ 50% 设计警钟；新增可证伪点 (BKN + 通用 embedding 是否足以支撑 5k+) |
| v12 | 2026-05-02 | [12] Component Composition (Knapsack) | Goal 4 Composer design-time pipeline 原型 (skill→retrieval→sandbox→ZCL)；OneFlow 折叠等价第二条边界（supervisor delegation 在 100+ 跨域 agent 被绑死）；新增"Design Goal 视角整合"章节按 0.8 路线图能力槽位重组 [8]–[12] 影响 |
