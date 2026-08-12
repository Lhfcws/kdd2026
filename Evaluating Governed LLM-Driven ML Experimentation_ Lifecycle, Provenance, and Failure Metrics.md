# Evaluating Governed LLM-Driven ML Experimentation: Lifecycle, Provenance, and Failure Metrics

## 背景与 Idea
随着 LLM 驱动的 ML 实验（闭环 autoML、自动建模）在企业中被采用，实验过程的可治理性与可审计性成为短板。现有评测多关注最终结果，缺少对实验全生命周期的治理度量。论文提出对"受治理的 LLM 驱动 ML 实验"在生命周期、来源记录（provenance）与失败模式上做系统性评估，把审计前置到实验运行过程本身。

## 核心方法
论文引入控制平面（control plane）并以 append-only ledger 记录实验事件，用 provenance completeness（来源完整性）衡量每个生命周期节点是否被 ledger 完整记录。在此基础上构建 failure taxonomy（失败分类法）刻画实验过程中的典型失败模式，并对比 governed trials 与 ungoverned trials 在可审计性上的差异。实验以 deepseek-v4-flash 作为 proposal manager 驱动实验循环。

## Performance 概括
在 1445 个 governed trials 上，6 个生命周期节点的 provenance completeness 均达到 100%。对照的 ungoverned trials 中，86%（43/50）完全没有 ledger 记录，几乎不可审计。结果说明治理平面能近乎完整地覆盖实验来源，而缺少治理时来源基本缺失。

## 论文价值与不足
价值在于给出可操作的治理与来源完整性度量，对 autoML/AI 实验审计与合规有直接现实意义。不足在于实验规模与所用模型范围有限，且 governed 与 ungoverned 的对比可能受到配置差异而非治理本身的干扰，因果性需更严格控制。

## 工业界落地可行性评估
落地场景明确：企业内部的 autoML/实验平台可借鉴其 ledger 与来源完整性审计思路，把实验过程变为可复现、可追责的记录。整体可行性较高，属于"监管理念 + 轻量工程"的可直接复用方案。
