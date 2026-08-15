# Outcome Is Not Enough: Trace-Based Lifecycle Evaluation for Trustworthy Agentic Economic Systems

## 背景与 Idea
agentic 经济系统常以标量业务结果（收入、转化、支出效率、延迟）评估，但这些信号在 agent 处于隐藏状态并改变其所评估的系统时可能不可信。作者以双酒店定价 benchmark 研究此失败：一个学习定价 agent 可达到合理 RevPAR，却未能保持规则基准的收益管理纪律。由此提出 discipline stability（纪律稳定性）——策略在部署可观测状态下既保持标量结果、又保持基准的行为轨迹结构。

## 核心方法
两酒店定价模拟器：Hotel A 为学习 agent，Hotel B 为固定收益管理（RM）对手（确定性规则，隐藏库存与规则）。信息体制分 NC（无对手价）、CA-lag（滞后市场价）、Oracle。discipline-stability 协议五步：定义基准纪律 → 限定可部署观测体制与隐藏变量 → 从失败诱导 trace 诊断（用 D1 距离与 Jensen-Shannon 散度度量价格/bid 分布偏离）→ 消融分离弱优化/隐藏状态/确定性坍缩/分布修复 → 测试基准访问减少后纪律是否持续。对比 reward-only PPO 变体（负对照）与 Trace-Prior teacher、corrected-history student，并在第二域 hidden-budget bidding 上验证。

## Performance 概括
reward-only PPO 变体失败于基准 trace：PPO 的 RevPAR A=93.55、Trace-Prior teacher=108.06（后者 D1=0.0165、D_JS≈0.0000）。容量不对称（QA=120, QB=100）下 Trace-Prior RL 提升 RevPAR +0.764（95% CI [0.125, 1.403]）。隐藏状态机制：argmax 预测逐步准确率 78.14% 但聚合对齐更差；采样把步准确率降至 69.50% 却改善 D1（0.0323→0.0183）。第二域 hidden-budget bidding：outcome-only 激进出价 value/step=0.374 且纪律差（D1=0.986），trace-prior 采样保留专家 trace 最佳（pacing gap 0.059，D1=0.0130）。

## 论文价值与不足
价值在于将"代理失败/奖励误设"框架化为可度量的生命周期评估协议，揭示"标量结果相同但行为纪律失败"的现象，并给出监控栈（标量 KPI + 业务分解 + trace 分布 + 状态切片 + 持续性测试）。不足在于：模拟器为合成、未校准真实酒店数据；Hotel B 是固定基准而非学习对手；bidding 任务紧凑；trace prior 质量受限于基准 trace（若基准本身有偏/串谋则保留错误纪律）。

## 工业界落地可行性评估
直接面向定价/bidding 等经济 agent 的生产监控（"分数提升了，但行为纪律是否保持"），论文给出可部署的监控栈与持续测试建议，落地可行性中等——需真实数据校准后才可用于生产治理。
