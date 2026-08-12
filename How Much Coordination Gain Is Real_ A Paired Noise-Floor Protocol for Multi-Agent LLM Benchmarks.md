# How Much Coordination Gain Is Real? A Paired Noise-Floor Protocol for Multi-Agent LLM Benchmarks

## 背景与 Idea
多 agent LLM 协调架构常以小幅基准提升宣称某架构优于另一架构。作者质疑：当两种协议在 trial 0（协调机制逻辑上尚未激活）时 API 输入配置等价，它们的配对差异究竟有多大？论文用 ET-MCP 作为配对测量基底，提出"协调噪声地板"（coordination noise floor）协议，主张它应成为任何协调架构性能声明必须跨越的发布门禁，用以区分真实的协调增益与单纯测量噪声。

## 核心方法
在 τ²-bench retail 上使用 Claude Haiku 4.5，对 no_coord / pull / intercept 三种协议做 n=100、双 seed 的配对测量，并通过 SHA-256 字节审计确认 trial 0 的请求等价。论文定义 coordination-active passₖ（仅在协调存储非空时才计入统计），并给出三个运行时告警作为发布门禁。整体设计强调同模型、同输入下的配对比较，以隔离协调机制的边际贡献。

## Performance 概括
trial 0 的干净配置等价对比（no_coord vs intercept）在两 seed 合并下为 +5pp（Wilson 95% CI [-2, +12]，不显著）；最大单 seed 对比 +18pp 在第二 seed 未复现（−3pp）。双 seed 配对差距包络为 [-3, +18]pp，合并上界 Wilson CI ≲15pp。7/10 近期多 agent 协调架构报道的提升低于该本地噪声地板。协调活跃子集（informative pairs 仅 8–17）在当前功效下无可检测效应，而非干净的零效应。

## 论文价值与不足
价值在于首次给出"同模型配对噪声地板"作为协调架构声明的发布门禁，并指出单 seed 显著结果（p_corr=0.012）在第二 seed 即瓦解的实例，凸显多 agent 基准的脆弱性。不足是主结论仅限 Haiku 4.5 + retail 域，跨模型/域探针显示地板随能力与域变化（Haiku airline ≈0pp，Sonnet retail 方向翻转）；协调活跃子集样本过小，架构对决属于功效不足下的非结果而非确定性结论。

## 工业界落地可行性评估
中等可行。其"发布前同模型配对地板 + coordination-active passₖ 门禁 + 运行时告警（CUSUM/R2/R3）"可直接用于多 agent 系统的上线前评估与漂移监控，尤其适配 Amazon Bedrock AgentCore / SageMaker Model Monitor 之类平台。但在各自模型与业务域上需重新跑配对协议以得到域相关的 MDE（最小可检测效应），否则门禁阈值不可直接套用。
