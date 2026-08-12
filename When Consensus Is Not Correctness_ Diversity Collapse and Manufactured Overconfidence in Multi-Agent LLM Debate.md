# When Consensus Is Not Correctness: Diversity Collapse and Manufactured Overconfidence in Multi-Agent LLM Debate

## 背景与 Idea
multi-agent LLM debate 被广泛认为能提升答案质量，且 agent 之间的一致（consensus）常被当作"答案正确"的置信信号（如 "debate until consensus"）。本文指出这一直觉在一个重要且可预测的 regime 下是错的：辩论产生的一致性是其自身内生的（endogenous），不能用作校准信号，反而会制造过度自信（overconfidence）。

## 核心方法
论文用经典的 equicorrelation 方差恒等式建模：随着 agent 互相读取彼此的输出，跨 agent 相关性 ρ(t) 趋向 1，即"多样性崩溃"（diversity collapse），此时真实误差与观测到的一致反向变化——误差不降而置信却涨。基于此提出 certification-first 的三档方案：Prevent（隐藏同伴结论）、Detect（对一致轨迹做选择性预测打分）、Certify（用 split-conformal 的集合替代一致性）。核心的 Certify 组件称为 Affirm。理论贡献为 Theorems 4.1–4.3 与 5.4–5.5。

## Performance 概括
在 15 个任务的受控研究中，跨 agent 相关 ρ̄ 从首轮 0.53 升至末轮 0.96（30/30 个条件都成立）；末轮一致度 κ̄(T)=0.997，置信被压缩到 [0.90,1.0]（均值 0.96），而准确率却跨度 [0.57,0.99]。过度自信 gap 与残差呈仿射关系，斜率 0.82、R²=0.96；准确率变化与 ECE 变化的相关系数 r=−0.46。单看一致性的 AUROC 仅 0.49–0.58（几乎无法检测错误），而基于一致性的停止规则误覆盖高达 18–47%。实验模型为 GPT-4o-mini 与 QwQ-plus，基准含 CommonsenseQA 与 GSM8K。

## 论文价值与不足
价值在于用理论+实证清晰揭示 debate 一致性的"过度自信制造"机制，并给出可证明覆盖（split-conformal）的 Certify 解法，立场鲜明且可操作。不足在于实验主要限于 QA 类基准，对自由生成、工具调用等更开放场景的验证还需补充；Certify 的集合可能偏大而影响可用性。

## 工业界落地可行性评估
落地性明确：任何用多 agent 辩论或自一致性（self-consistency）来估计置信、决定停止的场景，都应改用 conformal 的 Certify（Affirm）而非"一致即停"。这能避免在高风险决策中因虚假一致而过度自信，属于低风险、高收益的工程替换。
