# When Should Active RAG Retrieve? A Budget-Aware Evaluation of Utility, Calibration, and Cost

## 背景与 Idea
Active RAG 让 agent 自主决定何时检索，但"何时该检索"长期缺乏统一评估。作者将其重构为"预算效用估计（budgeted utility estimation）"问题：在给定检索预算下权衡效用、校准与成本，而非简单依赖"相似度低于某阈值就检索"的启发式。

## 核心方法
在 HotpotQA、2Wiki、MuSiQue 三个多跳 QA 数据集上，用 Qwen2.5-1.5B（每数据集 2,000 例）评估 Active RAG 的检索决策。定义 benefit（检索带来的准确率提升）与 harm（错误检索带来的损害），并以"名义阈值 50%"作为常见触发启发式，检查其在不同 split 上的校准性（是否出现本该检索却未检索、或反之的违规）。

## Performance 概括
HotpotQA 上 no-RAG 准确率 15.4%，always-RAG 43.4%，检索 benefit 31.9%，harm 4.0%。nominal 50% 阈值在 28.6%–65.7% 的不同 split 上出现校准违规，说明固定阈值触发检索并不稳健，需预算感知的效用估计。

## 论文价值与不足
价值在于把 Active RAG 的触发决策从启发式提升为可度量的预算效用估计，量化了 benefit/harm 与校准缺口，为多跳 QA 检索策略提供了评估基准与"效用—成本"权衡视角。不足在于仅用单一小模型 Qwen2.5-1.5B，大模型与其他任务（如代码生成、agent 工具调用）的泛化未验证；benefit/harm 的定义依赖具体数据集标注，跨域迁移需重新标定。

## 工业界落地可行性评估
RAG 是企业知识问答与 agent 工具检索的核心组件，预算感知的效用/成本权衡直接对应生产中的延迟与 API 成本管控，本文评估框架可指导检索触发策略的选型与阈值设定，落地可行性高。
