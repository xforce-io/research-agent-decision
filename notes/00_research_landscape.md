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
