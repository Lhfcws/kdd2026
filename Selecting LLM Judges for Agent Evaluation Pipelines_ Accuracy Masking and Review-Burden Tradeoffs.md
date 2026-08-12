# Selecting LLM Judges for Agent Evaluation Pipelines: Accuracy Masking and Review-Burden Tradeoffs

## 背景与 Idea
在 agent 评估管线中用 LLM 充当 judge 时，judge 模型的选择会系统性地"掩盖"（accuracy masking）某些失败模式，使整体评估结果失真。作者聚焦两类评估：side-effect（副作用/策略违规）与 task-success（任务是否完成），研究不同 LLM judge 在这两类上的准确率差异，以及由此带来的人工复核负担（review burden）权衡，揭示"选哪个 judge"本身就在悄悄改写评估结论。

## 核心方法
研究基于 AgentRewardBench，使用 3 个 agent、3 个 web 基准、5 种 judge 条件，对比 judge 在 side-effect 与 task-success 两类标签上的准确率差异，并量化两种典型行为的影响："高召回 judge（过度标记 overflagger）"会放大复核成本，"静默漏判 judge（silent misser）"则让失败逃过评估。

## Performance 概括
在 side-effect 类错误上，不同 judge 的准确率差异高达 45–50pp；而在 task-success 类上差异仅为 1.6–4.4pp，说明 judge 选择对副作用类评估的影响远甚于任务完成类。GPT-4o 与 Llama-3.3-70B 表现为 high-recall overflagger（高召回、过度标记）；GPT-4o-mini 是 silent misser，漏报率 FNR 达 0.64–0.77；Claude 3.7 的误判随 agent 不同而变化（agent-dependent）。

## 论文价值与不足
价值在于量化了"judge 选择本身会改变评估结论"这一被忽视的效应，给出 accuracy masking 与复核成本之间明确的权衡关系，对评估管线设计有直接指导意义。不足是实验基于特定基准与有限 judge 集合，未给出普适的 judge 选择准则；side-effect 的 ground truth 标注成本也可能限制结论向其他域外推。

## 工业界落地可行性评估
高度可行。论文可直接指导生产中的 agent 评估管线设计：对高风险的副作用类失败应优先选用 high-recall judge 并承受更高复核成本，对任务完成类则可选用更省成本的 judge。这种权衡框架可立即用于落地的评估系统，以提升对策略违规的检出率并合理分配人工复核资源。
