# When Aggregate Alignment Misleads: Auditing Agentic Policy Repair Without Per-State Expert Actions

## 背景与 Idea
agentic 系统越来越多地编辑/修复决策策略，但缺少 per-state 专家动作标签时难以评估修复质量。作者以酒店定价为受控测试床，研究"诊断反馈驱动的 policy repair"：编辑器仅收到区域级诊断（价格分布与基准在 time/inventory/market 上的差异），看不到基准动作、源码、奖励或留出结果，只能对 target-action 表做受限编辑。核心风险是"聚合对齐误导"——策略可在边际动作分布上像基准，却在轨迹级组成与收入上失败。

## 核心方法
策略是可由 time×inventory×market 索引的 target-action 表；基准为固定 RM 规则（行为参考，非最优证书）。编辑器收到 12 区域诊断，做矩形整数位移编辑。对比三类编辑器：diagnostic projection（直接投影，强廉价基线）、tree-search（非语义搜索，压力测试）、multi-restart LLM editor（DeepSeek-V4-Flash，125 restarts × 20 iters，共 2,500 候选）。在 5,000 条留出生产 episode 上审计，采用多度量：RevPAR/occ/ADR + pooled D1 + episode D1 + reference-state D1，外加 shuffled-diagnostic placebo 与 compute-matched 非语义 proposer 对照。

## Performance 概括
LLM editor 将 RevPAR 从 102.75 提至 108.47（基准 108.75），paired gap -0.276（95% CI 跨零 [-0.692,0.146]）；episode-composition distance 从 1.153 降至 0.609（最强非基准结果），但 reference-state D1 1.197 差于基线 1.153。tree-search 达到最强 pooled D1 0.214、ref-state D1 0.328，却 RevPAR 跌至 98.91。diagnostic projection 达 RevPAR 107.90。非语义 proposer（≤2,500 评估）落后 8.77–14.57 RevPAR；shuffled diagnostic 崩溃至 RevPAR 94.30（破坏区域—错误对应后 pooled 对齐仍看似良好）。

## 论文价值与不足
价值在于揭示"聚合对齐误导"——policy 在边际动作分布上像基准，却在轨迹级组成与收入上失败；提出多粒度行为审计 + 诊断 placebo 对照协议，证明 LLM 编辑能用区域诊断产生近基准闭环收入（非仅靠重启搜索或提示格式）。不足在于：仅单一模拟器与单一基准策略；主 LLM 结果仅一次生产运行（DeepSeek 无 seed，运行级方差未测）；LLM 在 reference-state 上仍有显著残差距离；target-action 表仅为一种策略表示。

## 工业界落地可行性评估
论文将协议定位为"离线 policy-repair 助手"而非自主部署决策者：LLM 提补丁，外部校验/评分/选择/审计，需留出审计后才影子/金丝雀部署。工程落地路径清晰（dashboard 需同时报 outcome 与多层行为距离），落地可行性中等偏高。
