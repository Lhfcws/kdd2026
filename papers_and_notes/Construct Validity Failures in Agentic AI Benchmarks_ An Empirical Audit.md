# Construct Validity Failures in Agentic AI Benchmarks: An Empirical Audit

## 背景与 Idea
agentic AI benchmark 常被当作模型能力的排序依据，但这些榜单是否真的在测量它们声称的"构念"（construct）却很少被检验。论文用心理测量学中的 construct validity（构念效度）方法，对主流 benchmark 做实证审计，揭示排名背后的可靠性问题。

## 核心方法
论文选取 5 个 benchmark（τ-Retail、τ-Airline、SWE-bench Verified、GPQA Diamond、MMLU-Pro），在 15 个模型上计算跨 benchmark 的 Spearman 相关与排名反转，检验收敛/区分效度。据此提出五个 desiderata（D1–D5）：显式构念定义、收敛效度、区分效度、按构念报告分数、披露排名稳定性。

## Performance 概括
跨 benchmark 的 Spearman 相关均值 ρ=0.67（范围 0.10–0.92），22% 的模型对存在排名反转；τ-Airline × MMLU-Pro 的 ρ 仅 0.10（p=0.85，n=6），几乎无关。推理专用模型（o1、o3-mini、o4-mini）在 GPQA/MMLU-Pro 进入前 5，但在 tool-use 基准跌至第 6–12 名；o4-mini 在 GPQA 排第 2（81.4%）、τ-Retail 第 6（71.8%）、τ-Airline 第 9（49.2%）。最终没有任何一个 benchmark 满足全部五条 desiderata。

## 论文价值与不足
价值在于用严谨的心理测量学框架暴露 benchmark 的构念效度缺陷，推动更规范、更可解释的评测实践。不足在于这是基于 15 个模型、5 个 benchmark 的某一时点快照，结论会随新模型与新榜单出现而变化，需要持续审计。

## 工业界落地可行性评估
落地性强：企业在做模型选型或内部评测时，应关注构念效度而非盲信单一榜单，避免被高相关但低区分度的分数误导。论文的 D1–D5 可直接作为评测设计的自检清单。
