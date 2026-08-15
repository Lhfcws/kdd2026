# Verifying Agents in Rubric-Graded Environments

## 背景与 Idea
在按评分细则（rubric-graded）打分的场景中，验证 agent 输出是否满足每条细则，是可信 agent 落地的关键一环。现有 verifier 往往成本高、能力覆盖不全，难以在真实任务中既准又省地做验证。

## 核心方法
论文提出 BankerVerifierBench（BVB）：包含 3204 条人工判定的 rubric criteria，覆盖 21 个投行任务（源自 BankerToolBench）。从 rubric 反推出九大 verifier 能力，并提炼三条设计原则——P1 Reactive verification、P2 Environment alignment、P3 Domain guidance——将其实现于开源 verifier Gandalf。Gandalf 还被泛化到 OpenClaw（OPC）benchmark 检验跨域能力。

## Performance 概括
Gandalf 在 9 项能力中的 7 项领先且处于 Pareto 最优；其最便宜配置达到 F1 0.633、成本仅 $42，反而超过最贵 baseline 的 F1 0.538/$414。泛化到 OPC 时 F1 达 0.951，超过次优 7.5 个 F1 点。消融显示：移除 P3（judge guidance）F1 从 0.660 降至 0.649、成本 $64→$73；移除 P2+MCP 时 F1 基本不变（0.661 vs 0.660）但成本上升 60%（$64→$102）。

## 论文价值与不足
价值在于给出低成本、可泛化、且 Pareto 占优的 verifier 设计与可操作原则，对金融/合规审核有现实意义。不足在于评测聚焦投行域，跨域（如医疗、法律）通用性仍需验证；原则是否普适也存在边界。

## 工业界落地可行性评估
落地性强：金融与合规审核场景可直接采用 Gandalf 类 verifier 替代昂贵的人工/大模型评审，在保持质量的同时显著降本。属于"开源 + 原则化"的高可行性方案。
