# A Framework for Evaluating Agentic Skills at Scale

## 背景与 Idea
现实世界中的 agentic skills 数量庞大且持续演进，但缺乏大规模、可复现的评测框架来横向比较不同 agent-model 配置。论文（来自 Tessl）提出一套规模化评测框架，把"技能"作为评测单元，覆盖真实的技能与任务分布。

## 核心方法
框架评估 500 个真实世界 skills，自动生成约 1000 个任务，并覆盖 19 个 agent-model 配置（开源与专有并存）。整体由 environment engineering、task generation、validation agents 三部分构成，并支持 human-in-the-loop 校验。评测指标聚焦 instruction-following（指令遵循）与 goal-completion（目标完成）两个维度。

## Performance 概括
在公开结果图（图 1）中，Opus 4.8 以 88.0 最高，Opus 4.7 为 87.7；GLM 5.1 达到 85.0，表现接近头部。Kimi K2.6、MiniMax 2.7、Qwen3-Coder-Next、Gemini 3.1 Flash Lite 等约在 57–60 区间；Nemotron 更低。结果刻画了不同模型配置在规模化技能评测下的明显梯队。

## 论文价值与不足
价值在于提供大规模、可复现的技能评测方法论与可比排行榜，对 agent 选型有直接参考意义。不足在于任务由框架自动生成，可能与真实生产分布存在偏差；human-in-the-loop 校验虽然提升质量，但带来较高成本，难以无限扩展。

## 工业界落地可行性评估
落地性强：企业可用该框架对内部 agent skills 做常态化、规模化的评测与回归，支撑模型/agent 选型决策。框架化、可复现的特性使其易于集成进 CI/CD 式的评测流水线。
