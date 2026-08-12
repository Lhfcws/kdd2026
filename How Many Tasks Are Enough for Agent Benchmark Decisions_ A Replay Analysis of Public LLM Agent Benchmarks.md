# How Many Tasks Are Enough for Agent Benchmark Decisions? A Replay Analysis of Public LLM Agent Benchmarks

## 背景与 Idea
agent benchmark 常在跑完全部任务后比较两系统，但昂贵评估让"部分运行"很诱人。作者指出：任务比例本身不能说明部分运行是否支持与完整 benchmark 相同的成对结论。核心概念是"最小充分任务预算（minimum sufficient task budget）"——在给定阈值、覆盖规则、未解决比较上限下，能可靠复现完整基准成对决策的最小任务比例。

## 核心方法
重放（replay）SWE-bench、AppWorld、tau-bench 的已完成的公开任务级记录：固定任务集/系统/结果，仅改变在预算耗尽前观察到哪些任务。定义"充分预算"需同时满足三目标：决策误差目标（条件 false-accept / false-reject ≤ 5%）、任务组覆盖（repository / 难度 / domain 必须按比例出现）、未解决比较率（defer ≤ 25%）。对比多种策略（uniform / group-stratified / cost-aware / coverage-aware bootstrap），在 0/5/10 pp 改进阈值下扫描 5% 预算网格，并做 bootstrap cutoff、样本数、分配、经典配对检验等敏感性检查。

## Performance 概括
在 0 pp、5% 预算网格上：AppWorld 首次达标于 15%，tau-bench 25%，SWE-bench Verified 90%；SWE-bench Lite 在 95% 前均未达标（primary coverage rule 下）。原因：SWE-bench Lite 在 0 pp 时即便误差/覆盖达标，仍有 55.92%（25% 预算）至 27.25%（95% 预算）比较未解决；SWE-bench Verified 在 25% 预算时 93.64% 未解决。tau-bench 的 cheap-first 排序省成本（仅用 11.51% 测试成本）却 100% 覆盖失败并给出错误成对结论。

## 论文价值与不足
价值在于识别"部分评估报告"的失败模式（隐藏错误决策、缺失任务组、过多未解决比较），给出可操作的最小充分预算概念与重放方法论，并公开代码/种子。要点是：部分评估必须声明阈值、任务选择、覆盖规则、决策规则与未解决比较数。不足在于仅基于已完成记录的排列重放，不估计未来任务表现；依赖公开元数据的分组（repo/难度/domain），未涵盖语义/失败模式聚类；自适应或主动任务选择可能改变预算。

## 工业界落地可行性评估
直接服务 benchmark 使用者与采购/部署决策，论文建议的部分评估规范可显著降本同时保持结论可信度，作为评估规范与开源重放工具落地可行性高。
