# Equilibrium-Level Harm in Multi-Agent AI Systems: Why Individual Evaluation Fails

## 背景与 Idea
多 agent 系统通常采用个体评估来判定安全性，但个体理性的 agent 在战略互动中可能产生集体有害的纳什均衡——论文称其为 equilibrium-level harm（均衡级危害）。这类危害在单个 agent 视角下完全"合格"，因此个体评估会系统性地漏检。

## 核心方法
论文形式化定义了 equilibrium-level harm，并给出 Theorem 3.1：仅基于固定非战略输入的个体评估，必然存在"个体通过却产生 equilibrium-level harm"的游戏。为验证，构造合成博弈 Shared-Engagement Contest（K=4，p=(0.50,0.30,0.16,0.04)，κ=0.05）。论文还归纳出三个结构性风险预测因子：objective correlation、environmental constraint、以及 skewed outcome distributions with demographic correlation。

## Performance 概括
在 Wang et al. 的政治排斥实验中，Qwen3-4B 的排除区为 0.510%，而 Qwen3-4B-Thinking 为 4.535%（约 8.9 倍）；跨家族上 4B 与 7B 的差异约 17.7 倍。合成实验中单 agent 覆盖率 0.079（通过阈值 τ=0.05），竞争均衡（n=4）降至 0.033、排除率 0.25；引入干预 γ=0.10 可恢复覆盖，但总参与仅增加 0.4%，说明危害难以被低成本修补。

## 论文价值与不足
价值在于从理论上证明个体评估的系统性盲区，并提供可识别的风险因子，对多 agent 安全审查有方法论意义。不足在于合成博弈较简化，真实多 agent 生态（如市场、社交模拟）中的验证仍有限，定理假设在开放环境下可能弱化。

## 工业界落地可行性评估
可落地于多 agent 协作、市场机制与撮合系统的安全审查，尤其对"个体合格但系统有害"的隐蔽风险有警示价值。建议在 agent 上线前增加均衡级/对抗性评测环节，而非仅做单 agent 测试。
