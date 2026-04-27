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
