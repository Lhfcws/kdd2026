# NRT-Bench: Benchmarking Multi-Turn Red-Teaming of LLM Operator Agents in Safety-Critical Control Rooms

## 背景与 Idea
在安全关键的控制室（如核电厂）中，LLM operator agent 需要长期值守并应对对抗性输入。单轮评测难以反映真实威胁，论文提出对控制室里的多角色 operator agent 做多轮 red-teaming 评测，关注其能否维持关键安全功能。

## 核心方法
NRT-Bench 模拟核电厂控制室，设定 5 角色 operator team 与 6 个 CSFs（关键安全功能，Critical Safety Functions）。评测以多轮 red-teaming 方式施加攻击会话，并采用 judge-free 的主指标 ASRCSF（违反关键安全功能的攻击成功率），避免依赖主观评审带来的偏差。

## Performance 概括
在 replayed attack sessions 中，8.7%–12.1% 会导致至少一个 CSF 丢失。在 149 个会话里，50 个（33.6%）至少 breach 一个模型，但没有一个会话能 breach 全部四个模型，说明不同模型在被攻破的薄弱环节上有所差异，没有模型在所有维度上都安全。

## 论文价值与不足
价值在于把多轮对抗与 judge-free 指标引入安全关键域评测，更贴近真实威胁模型，对高风险场景有警示作用。不足在于环境为模拟控制室、攻击样本规模有限；judge-free 指标虽客观，但可能漏检需要语义理解的复杂攻击。

## 工业界落地可行性评估
可落地于工业控制、关键基础设施的 agent 安全测试，价值高。但需针对具体领域（电网、化工、交通）定制 CSFs 与攻击剧本，工程化与领域知识门槛较高，离即插即用尚有距离。
