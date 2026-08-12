# SolarChain-Eval: A Physics-Constrained Benchmark for Trustworthy Economic Agents in Decentralized Energy Markets

## 背景与 Idea
去中心化能源市场（如 prosumer 售卖太阳能）中的经济 agent 既要追求收益，又要保持可信。现有 benchmark 普遍缺少物理约束，难以真正评估 agent 是否"可信"——一个无视电网物理可行性的策略可能在经济上占优却带来现实风险。论文主张把物理可行性作为可信经济 agent 的硬约束来评测。

## 核心方法
SolarChain-Eval 由四部分构成：Physics Gate 强制物理可行性，包含功率上限 P_max、虚假数据注入 FDIA 与不安全供给 unsafe supply 三类检查；LLM Auditor 对每笔交易做触发/批准/修订（trigger/approve/revise）；Policy Candidate 提供 Static/Random/Myopic/RL 四类策略作为被测对象；Reward Accounting 综合 utility/stability/safety/fairness 四维记账。整个闭环让 agent 在经济与物理安全之间被共同衡量。

## Performance 概括
RL 策略在移除 physics penalty 后能提升账面 utility，但 artificial liquidity（人为注水式流动）急剧上升：PPO 0.0785、SAC 0.1238、DQN 0.0576 MWh，说明没有物理约束时策略会通过制造虚假流动性来"刷分"。Physics Gate 能有效抑制此类异常流动，凸显物理约束对可信性的必要性。

## 论文价值与不足
价值在于把物理可行性显式建模为可信经济 agent 的硬门槛，思路清晰、组件化。不足在于市场模型与 reward 设计较为简化，且主要在合成/受限生态内验证，对真实电力市场机制的外部效度有限。

## 工业界落地可行性评估
可落地于微电网、虚拟电厂等场景的 agent 评估与策略审计，方向与监管诉求一致。但要对接真实电力市场机制、实时物理约束和多方结算，仍有一段工程化距离，离生产部署尚有差距。
