# Research Landscape

> This document tracks the evolving state of the field as researcher reads papers.
> Researcher maintains this file; human reviews changes via PR diff.

## Overview

Decision agent 技术路径综述。关注决策架构的设计模式、技能管理机制，以及多智能体协同完成长期任务的方法。

## §1 Decision Agent Architecture

### §1.1 Co-Evolving Multi-Agent Systems

- **COS-PLAY** [1] (Wu et al., 2026) — 将决策智能体（A_D）和技能库智能体（A_S）解耦并用 GRPO 联合训练，通过 7–25 轮协同进化达到收敛。8B 模型（Qwen3-8B）在四款单人游戏上平均奖励超 GPT-5.4 25.1%；Diplomacy 多人游戏优于 Gemini-3.1-Pro 8.8% [1: §5.1]。关键约束：技能库分布与决策策略分布必须对齐——错位版本反而降低性能 [1: §5.2]。技能可复用性：每技能平均 12–49 次实例化，2–6 次合约精化 [1: §5.3]。
  - *builds-on* Voyager (Wang et al., 2023) — COS-PLAY 将 Voyager 的外部技能库扩展为一个独立训练的技能库智能体 [med]
  - *competes-with* SkillRL (Xia et al., 2026), SCALAR (Zabounidis et al., 2026) — 两者同样将技能学习与 RL 结合，但技能库均为静态基础设施而非独立训练的协作智能体 [med]
  - *relation to thesis* — 直接实例化"multi-agent workers team"架构；技能库智能体跨 episode 无状态运行（无持续上下文），为决策智能体独立部署提供初步支持证据 [1: §Relations]

> *Emerging sub-cluster: Graph-based Test-time Routing* — 在推理时通过有向图结构进行智能体选择与消息传递，无需 RL 训练。GoA 不适配 §1.1（Co-Evolving），放置于父桶 §1；建议增设 §1.2，详见 `contradictions.md §Proposed taxonomy extension`。

- **GoA** [2] (Yun et al., 2026, ICLR 2026) — 将多智能体协作建模为有向图，分四阶段执行：①元 LLM 从模型卡采样 top-k 相关智能体（节点采样）；②各智能体互相评分生成有向相关权重边（边采样）；③双向消息传递（高相关节点→低相关节点，再反向）；④图池化输出（GoAMax 取最高连接节点响应，GoAMean 由元 LLM 加权合成）。GoAMax（3 agents）在 MMLU 79.18、MMLU-Pro 54.78、MedMCQA 60.04 上超越 MoA（6 agents）；GoAMean 在 GPQA 40.54、MATH 73.12 上超越 MoA；推理成本降低约 58%（11 vs. 19 LLM calls，19k vs. 56k tokens）[2: §4.2, Table 2]。关键机制：相关性有向消息传递——反转流向造成最大性能下降（–2.60 MMLU-Pro，–5.05 GPQA）[2: §4.4, Table 5]。
  - *competes-with* [1] (COS-PLAY) [low]: GoA 为零样本推理时框架，COS-PLAY 为带 RL 训练的长时序任务架构；均采用多智能体协同，但操作层级完全不同（QA 推理路由 vs. 游戏任务联合进化）
  - *competes-with* MoA (Wang et al., 2024) [high]: GoA 证明 MoA 是其退化特例（k=N、全连接均匀权重、mean pooling），并在所有主要基准上系统性超越 MoA [2: §3.3, §4.2]
  - *relation to thesis* — GoA 处理推理层智能体路由（单轮多域 QA），非决策智能体的多步任务执行；图式节点采样（元 LLM 按查询动态路由至专家子智能体）可映射为 openclaw→decision agent 调度链的轻量实现，与论题互补但不直接支持或反驳核心部署独立性主张 [2: §Relations]

> *Emerging sub-cluster: Single-Agent Future-Aware Planning* — 从理论上证明步进式推理（CoT/ReAct/Reflexion）等价于贪心策略，无法在长时序任务中替代前瞻规划。FLARE 不适配 §1.1（Co-Evolving）或图路由子簇，放置于父桶 §1；建议增设 §1.3，详见 `contradictions.md §Proposed taxonomy extension`。

- **FLARE** [3] (Wang et al., 2026) — 形式化证明步进式 LLM 推理（CoT、ReAct、Reflexion）等价于最大化局部代理分数的贪心策略，从结构上不足以支撑长时序规划 [3: §2, §3.2]。关键实证：第一步决策中，贪心策略在 55.6%、beam search 在 71.9% 的情况下选中局部最优但全局次优的"陷阱"动作，前瞻仅 23.6%；首步错误后单步恢复概率 5.4%（贪心）vs. 29.7%（FLARE）[3: §3.1, Table 3]。FLARE 通过 MCTS + 后向价值传播 + 轨迹记忆实现前瞻规划：CWQ Hits@1 71.8（FLARE）vs. 59.8（单步）[3: §5.2, Table 1]；LLaMA-8B + FLARE 频繁超越 GPT-4o + 标准推理，表明规划能力无法通过参数规模恢复 [3: §5.2]。关键局限：假设确定性转移、显式状态与规划时可用评估信号；单智能体评测，无多智能体协同证据 [3: Limitations]。
  - *orthogonal to* [1] (COS-PLAY) [med]: COS-PLAY 改进智能体"知道做什么"（技能库），FLARE 改进"如何在已知动作中选择"（规划机制）；两者解决正交缺口，组合系统需要二者 [1: §6, 3: §7]
  - *orthogonal to* [2] (GoA) [low]: GoA 处理智能体间通信路由（单轮 QA），FLARE 处理智能体内多步动作选择；系统层级完全不同，无竞争关系
  - *relation to thesis* — 为论题提供关键约束：若 decision agent workers 采用步进式推理（当前主流），在需要多步序列决策的任务上系统性失败；支持论题扩展——workers 内部规划机制必须具备前瞻性，而非贪心局部。不挑战多智能体框架，但对 openclaw×decision agent 协作栈提出单智能体内规划先决条件 [3: §Relations]

> *Emerging sub-cluster: Single-Agent Sufficiency Baseline* — 形式化证明并实证验证同质多智能体工作流可折叠为单智能体顺序执行而无精度损失，成本更低。[4] 不适配 §1.1（Co-Evolving）、图路由子簇或前瞻规划子簇，放置于父桶 §1；建议增设 §1.4，详见 `contradictions.md §Proposed taxonomy extension`。

- **OneFlow** [4] (Xu et al., 2026) — Proposition 1 形式化证明：在确定性工具副作用、基于可见历史的路由策略、共享随机性三个前提下，单 LLM 模拟器与多智能体系统的轨迹分布完全相同 [4: §3.1]。OneFlow（单智能体执行 + MCTS 双元 LLM 工作流搜索）在 6 个公开基准上匹配或超越 AFlow（多智能体），成本大幅降低：HumanEval $0.020 vs. $0.198（约 10×），HotpotQA F1 73.5 vs. 72.1，成本 $0.278 vs. $1.438（约 5×）[4: §4.2.1, Table 1–2]。单智能体 KV cache 复用将前缀成本从 Σ L_t 降至 Σ ΔL_t（渐进节省）[4: §3.1]。关键约束：等价结论仅适用于同质工作流（所有 agents 共享同一基础 LLM）；异质工作流（混合模型）性能上界由最强同质工作流决定，未超越 [4: §2, §4.2.3]；所有实验 temperature=0，形式化等价不覆盖随机采样 [4: §4.1]；最复杂基准为 TravelPlanner（单会话规划），多会话企业工作流未覆盖 [4: §4.1]。
  - *contradicts* thesis §1 (multi-agent necessity) [med]: 论题以多智能体 workers team 为架构核心；[4] 证明同质配置下多智能体结构无精度收益且增加成本——部分证伪；Decision Agent 的 BKN+ContextLoader+Dolphin+ISF 专化组件若构成异质配置则结论不适用 [4: §3.1; thesis §1]
  - *competes-with* [2] (GoA) [med]: GoA 将图结构多智能体路由作为性能路径；[4] 证明同质设置下复杂路由结构不必要——GoA 使用异质 7–8B 专家，[4] 结论不直接反驳，但将"单智能体基线"作为效率门槛施加于所有多智能体路由设计 [4: §3.1; 2: §3.3]
  - *extends* [3] (FLARE) [low]: FLARE 证明步进式推理规划不足；[4] 进一步表明多智能体任务分解工作流可折叠为单智能体顺序推理而不损精度——合并暗示性能天花板由规划机制决定而非通信拓扑，但此为合成推断，两篇论文均未明确 [3: §7; 4: §6]
  - *orthogonal to* [1] (COS-PLAY) [med]: COS-PLAY 使用独立训练 LoRA 适配器的异质双智能体，属 [4] 自身标准下必须保留多智能体结构的场景；[4] 明确指出单 LLM 模拟"无法捕获异质工作流" [4: §6; 1: §3]

> *Emerging sub-cluster: Catalog-Layer Tool/Agent Retrieval* — 在工具/智能体目录上构建类型化知识图，用类型加权 RRF 进行召回；与 §1.2（GoA 推理时图路由协同输出）层级不同——此处图是静态目录结构，用于 pre-call 选择哪些 worker 被激活。[5] 不适配既有子簇，放置于父桶 §1；建议增设 §1.5，详见 `contradictions.md §Proposed taxonomy extension`。

- **Agent-as-a-Graph** [5] (Nizar et al., 2025) — 将 MCP 服务器（agents A）和工具（T）建模为二部图 G=(A,T,E)，边记录 owner(t)=a；检索时对 A 与 T 在统一向量空间独立 top-N，再通过类型加权 RRF（αT/(k+r) 用于工具节点，αA/(k+r) 用于 agent 节点，k=60）重排，最后沿 owner 边遍历至 K 个唯一 agent [5: §3.1, §3.2, Algorithm 1]。LiveMCPBench（70 服务器/527 工具/95 问题）：Recall@5 = 0.85、nDCG@5 = 0.47，相对 SOTA ScaleMCP（0.74/0.40）+14.9%/+14.6%；最优类型权重 αA:αT = 1.5:1（轻度偏向 agent 覆盖），1:1 退化至 0.83/0.46，3:1 退化至 0.76 [5: §4.2, §4.4, Table 1, Table 3]。跨 8 种 embedding 模型（Vertex/Gemini/Titan v1-v2/OpenAI ada-3小-3大/MiniLM）平均 Recall@5 提升 19.4%（std≈0.02），Titan v2 上+28.8% [5: §4.3, Table 2]。检索来源分布：39.13% 直接来自 agent 节点，34.44% 来自工具节点经 owner 边遍历——两类型节点都对最终路由作出实质贡献 [5: §4.2.1]。
  - *builds-on* ScaleMCP / Tool-to-Agent Retrieval (Lumer et al. 2025c-d) [high]: 同组前作；[5] 在 ScaleMCP "拼接描述单向量"基线上引入类型化节点+类型加权 RRF，将多跳查询失败模式转为"显式类型节点+图遍历"修复 [5: §1, §2.1]
  - *builds-on* RRF (Cormack et al. 2009) [high]: 标准 wRRF 是出发点，[5] 改为分段式 type-conditioned 形式（Eq. 3），保留共享分母 k+r(e) 以维持类型内 RRF 排序性质 [5: §3.3]
  - *competes-with* [2] (GoA) [med]: 均为多智能体架构中的"图"，但层级不同——GoA 在推理时构图协调多 agent 输出（消息传递+图池化），[5] 在调用前对静态目录构图选 agent；二者可组合（[5] 选 K 个 agent → GoA 协同输出），双方均未引用对方（concurrent）
  - *competes-with* [4] (OneFlow) [med]: [4] 质疑同质多智能体的必要性；[5] 在多智能体框架内默认其价值并优化路由层——若 [4] 同质等价结论成立，[5] 的检索方案失去前提；若异质性成立，[5] 的类型化召回直接服务于异质 worker 选择
  - *competes-with* [1] (COS-PLAY) [low]: 二者占据同一"系统如何知道调用什么能力"的位置——COS-PLAY 通过协同进化"学到"答案，[5] 在静态目录上"检索"答案；研究问题正交（技能习得 vs. 工具选择），但架构角色重合
  - *orthogonal to* [3] (FLARE) [high]: 规划与检索是不同问题；[5] 假设查询/子步已给定，仅负责选 agent，[3] 关注子步如何正确生成；可在同一栈共存（planner→[5]→worker）
  - *relation to thesis* — 直接命中 ContextLoader/工具管理研究缺口。论题明确将"工具检索精确召回（向量 vs 语义结构化对比）"列为高优先级；[5] 的"类型化节点显式分离 vs. 拼接单向量"恰好是该对比，14.9% Recall@5 提升构成 BKN"逻辑/行动分离"语义解耦的实证支撑。最优 αA:αT=1.5:1 提示 ContextLoader 可暴露类型权重以在意图模糊时偏向父技能/父域覆盖。规模缺口仍开放：527 工具远低于企业目标的数千级别；架构无关性（8 种 embedding）支持可移植，但未在 ContextLoader 90%+ 上下文节省真正起作用的规模上验证 [5: §Relations]
