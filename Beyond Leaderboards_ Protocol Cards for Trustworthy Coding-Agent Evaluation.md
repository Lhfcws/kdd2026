# Beyond Leaderboards: Protocol Cards for Trustworthy Coding-Agent Evaluation

## 背景与 Idea
coding-agent 的评测高度依赖 leaderboard，但榜单排名对评测协议（protocol）极其敏感，而协议细节往往不透明。论文指出评测协议本身是一个隐藏变量，忽略它会导致不可复现、不可比的结论，因此提出 Protocol Cards 让评测过程可复现、可审计。

## 核心方法
论文引入 SNRfile 来记录评测协议中的噪声与配置，并用 clean-apply rate（补丁能否干净地应用到代码库）作为协议质量的客观指标。实证基于 7200 条 code-review records 与 1350 条 SWE-bench Lite records，比较 diff-only、structured、full-raw 三种协议粒度对信噪比与可应用性的影响。

## Performance 概括
从 diff-only 升级到 full-raw 协议，ΔSNRfile=0.007（p=0.002），信噪比显著提升。clean-apply rate 随协议完整度提高：diff-only 仅 8.0%，structured 提升到 50.4%，full-raw 进一步到 54.7%。说明协议信息越完整，评测越可信、补丁越可应用。

## 论文价值与不足
价值在于揭示"评测协议"是 coding-agent 评测中被忽视的隐藏变量，并给出可操作的协议规范（Protocol Cards）。不足在于 clean-apply 的提升在 structured→full-raw 阶段已趋于饱和（50.4%→54.7%），收益边际递减，且协议规范化本身需要额外标注成本。

## 工业界落地可行性评估
可直接落地：企业可在内部 coding-agent 评测中强制要求 Protocol Cards，统一协议描述，提升跨团队、跨时间结果的可比性与可信度。属于低成本、高杠杆的工程规范改进。
