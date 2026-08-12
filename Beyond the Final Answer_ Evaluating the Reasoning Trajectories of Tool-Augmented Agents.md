# Beyond the Final Answer: Evaluating the Reasoning Trajectories of Tool-Augmented Agents

## 背景与 Idea
现有 tool-augmented agent 的基准（如 GTA、m&m's、GAIA、MetaTool）几乎都依赖 Answer Match，只校验最终答案是否与 ground-truth 一致。作者指出这是不足的：两个 agent 即便达到相同的最终准确率，其推理过程可能天差地别——一个用最少步骤直达答案，另一个却充满冗余、幻觉或缺乏工具失败的适应能力。更关键的是，同一用户请求往往存在多条有效轨迹，逐一标注所有 ground-truth 轨迹代价高昂，因此需要一个无需参考答案、能评估推理轨迹质量的框架。论文由此提出 TRACE，从效率、幻觉、适应性三个常被忽视的维度评估 agent 的推理轨迹。

## 核心方法
TRACE（Trajectory-based Reasoning Assessment and Comprehensive Evaluation）面向 ReAct 式 agent，核心组件是 evidence bank：将每一步的 (action, input, observation) 累积为结构化证据，而非把整段非结构化对话直接喂给 LLM 评判，从而更稳定地衡量效率与幻觉。效率评估在 agent 给出正确答案后做 post-hoc 分析：用 LLM 从证据库中挑出推导答案所需的最小证据集 Emin，效率分数 Eff(T) = |Emin|/|Etotal|（越低表示冗余越多）。幻觉检测用 IsGrounded(th_t, E_{t-1}) 判断每一步 thought 是否可由此前证据支撑，偏离即记一次幻觉，分数取各步平均。适应性则在 tool 返回不可用错误时，评估后续步是否能合理切换到替代工具（二值评分）。整体无需预定义 ground-truth 轨迹，且可扩展到多智能体系统（为每个 agent 维护独立证据库）。

## Performance 概括
论文构造了 Meta-GTA 与 Meta-m&m's 两个 meta-evaluation 数据集（在 GTA、m&m's 上注入低效率/幻觉/适应性步骤并人工标注）来验证评估器本身。在 Meta-GTA 上，TRACE 相对朴素 LLM-as-a-Judge 在效率、幻觉、适应性三维度均有明显提升（如 Llama-3.1-8B 效率由 86.47 升至 90.03、适应性由 70.46 升至 85.28），且小开源模型受益最大。与 PIPA 的状态一致性指标相比，TRACE 在多种有效轨迹并存时方差更小、更稳健（Llama-8B 在 PIPA 上仅 24.94，而 TRACE 为 79.10）。在真实 GTA 任务上评测 9 个 agent（Claude、GPT-4.1、o3-mini、Llama、Mixtral、Mistral、Qwen 等）：o3-mini 幻觉率极低但适应性弱，GPT-4.1 与 Qwen-72B 总准确率相近（约 0.53 vs 0.52）却在效率/幻觉上各有短板；Figure 4 还显示输出 token 数、对话轮次与总准确率呈负相关，即越长轨迹越易失败。

## 论文价值与不足
价值在于把 agent 评测从"只看结果"推进到"看过程"，提出的三维度（效率/幻觉/适应性）切中工具型 agent 真实痛点，且 reference-free 设计与 evidence bank 机制让小模型也能胜任评判，降低了评测成本。不足方面：效率分数只衡量"所选工具中的冗余"，并不保证找到全局最优路径；幻觉定义偏宽松（只要用了证据库之外的内部知识就计为幻觉）；实验主要基于 GTA 单一基准，小模型（Mistral-7B）几乎无法产出正确答案导致很多指标缺失，泛化性结论需谨慎。

## 工业界落地可行性评估
该框架对需要真实部署 tool-augmented agent 的企业（如客服、数据分析、RPA）有较高实用价值：其 reference-free 特性意味着无需维护参考答案即可对线上 agent 轨迹做持续质量监控，证据库机制也便于定位冗余调用与幻觉步骤以优化成本。论文已开源代码（github.com/wonjoong-kim/TRACE），并指出 Llama-3.3-70B 评测耗时仅为大模型约 1/3，利于大规模低成本部署；可结合反馈机制与"accuracy × efficiency 加权"缓解幸存者偏差，适合作为 agent 上线前的回归评测组件。
