# PCBWorld: A Benchmark Environment for Engine-Grounded PCB Design Automation

## 背景与 Idea
PCB 设计自动化若只谈大模型"生成"，缺乏真实 EDA 引擎校验，就容易产出不可制造的结果。论文主张"引擎接地"（engine-grounded）的评测：让 agent 在真实布线引擎中行动，并用引擎判定结果是否合法可行。

## 核心方法
PCBWorld 基于开源 KiCad EDA engine，暴露 58 个 Python APIs，并封装为 Gym 风格环境，分三层：底层 C++ engine、中层 Gym MDP、上层 RL+LLM wrappers。PCBWorld-Bench 包含三个数据集族——D1（grid synthetic）、D2（gridless synthetic）、D3（679 块真实开源板），用八项引擎可校验指标衡量，主指标为 Clean Pass（CP，干净通过率）。

## Performance 概括
在 D2-test 上 PPO 的 CP 达 1.00、Potential 10.52，优于 Freerouting 的 9.76；D3-A 上 PPO CP 0.86 优于 Freerouting 0.80，但 D3-B 上 Freerouting 0.78 反超 PPO 0.45。LLM 方面，GPT-5.4 在 D2 的 CP 为 0.96，mini 0.58，nano 0.55；而在 D3-B 上所有 LLM 的 CP 均为 0.00。仅训练于合成板的 PPO 可零样本迁移到真实板，接近 rule-based routers。

## 论文价值与不足
价值在于提供引擎接地、可复现的 PCB 设计自动化基准，把"能否制造"变成可量化指标。不足在于 LLM 在复杂真实板（D3-B）几乎完全失败，显示大模型直接生成 PCB 仍有巨大差距；引擎所覆盖的 PCB 类型与约束也有限。

## 工业界落地可行性评估
可落地于 EDA 工具的自动化布线评估与算法对比，对 RL/搜索类方法尤为有用。但让 LLM 直接端到端生成可制造 PCB 目前尚不成熟，工业落地更可能先用于"辅助+引擎校验"的闭环，而非纯生成。
