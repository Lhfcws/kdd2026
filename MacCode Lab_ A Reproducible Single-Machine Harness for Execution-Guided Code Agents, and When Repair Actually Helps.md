# MacCode Lab: A Reproducible Single-Machine Harness for Execution-Guided Code Agents, and When Repair Actually Helps

## 背景与 Idea
代码 agent 评估正从一次性 leaderboard 走向随模型、提示词和 agent 组件变更而反复执行的工程实践，因此需要能在本地硬件经济地运行、并能指出 agent loop 中哪些环节带来可衡量价值的评估手段。论文提出 MacCode Lab，一个在单台 Apple Silicon 工作站上、基于 MLX 量化开放权重模型运行的执行引导式（execution-guided）代码 agent 评估框架，目标是把"评估一个代码 agent"本身当作可审计、可复现的系统工件来研究，而非简单地给模型打分。核心关切是：执行引导式 repair（错误反馈驱动的修复循环）到底在什么时候真正有用，以及仅以公开测试集（public tests）通过率来比较模型会产生多大偏差。

## 核心方法
MacCode Lab 是一个全本地、可断点续跑（resume-safe）的执行引导式评估 harness。它摄取缓存的 LiveCodeBench v6 前 200 个任务与 HumanEval 全部 164 个任务，使用一个按难度条件化的 UCB1 bandit 在 8 种 prompt 风格中选择，在隔离子进程中生成并执行候选程序，将失败归类为一个紧凑 taxonomy（syntax/assertion/name/timeout/wrong-answer 等），并将错误特定的提示送入 execution-guided repair loop。框架把每个任务的记录、运行状态与 bandit 状态持久化到磁盘，从而支持无人值守运行、运行中检视与运行后重打分。实验中两个模型（Qwen3.6-35B-A3B 的 3-bit MLX 与 Qwen2.5-Coder-7B 的 4-bit MLX）均在 M5 Max 128GB 单机上以 greedy 解码运行；关键设计是隐藏测试仅用于事后重打分，绝不进入生成或 repair 提示，以避免污染。

## Performance 概括
在 LiveCodeBench 上，repair 对 35B 模型将公开测试通过数从 89 提升到 113，但对 7B 模型没有净增益（公开测试 125→125，full protocol 下 repair 反而从 102 降到 99）。当用完整 hidden-test 协议重打所有被接受的解时，35B 仅丢失 3–7 个解（full protocol 从 88 升到 106），而 7B 丢失 23–26 个（102 与 99），即 public-to-hidden 衰减在两者间严重不对称。结果是：仅靠公开测试的比较偏向 7B，但在完整协议下差距收窄、且在 repair 条件下排序反转（106 vs 99）。HumanEval 因接近饱和，repair 对两模型最多只改变 1 题。Wilson 95% 置信区间约为 ±7 个百分点，作者明确不宣称跨模型排序具有统计显著性。失败 taxonomy 中 syntax 失败以 562 次居首，是明确的工程改进目标。

## 论文价值与不足
论文的主要价值在于提供了一套可复现的本地评估仪器，并给出"repair 是否有用取决于具体模型而非仅取决于任务"的实测证据，以及 public-test 分数与 full-protocol 结果可能系统性分歧的警示。不足在于：研究仅覆盖两个模型、单一硬件/量化/MLX 版本，LiveCodeBench 用的是确定性前缀 200 题而非分层随机采样，结果为单 seed，且仅支持单文件 Python 任务（functional/stdin/call 三种分发）；repair 同时改变了信息feedback与额外采样次数，未通过配对消融分离"错误提示、额外样本、bandit 状态"各自的因果贡献。

## 工业界落地可行性评估
单台 Apple Silicon 工作站即可作为可审计、可重放的代码 agent 评估工具，一个研究员可在一天内运行、检视并复现该评估，对经济频繁做增量评估的工程团队具有现实吸引力。论文明确建议：本地 harness 在报告执行类分数时，应把 public-test 结果与其后针对全协议的模型特定重打分一并给出，因为 public-only 评分会对不同系统引入不同偏差。其释放的 harness、冻结运行记录与重验证脚本可直接支撑结构化编辑、上下文缓存和自适应 repair 策略的可复现实验。
