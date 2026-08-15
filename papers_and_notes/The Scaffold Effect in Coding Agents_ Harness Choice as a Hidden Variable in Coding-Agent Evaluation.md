# The Scaffold Effect in Coding Agents: Harness Choice as a Hidden Variable in Coding-Agent Evaluation

## 背景与 Idea
公开编程智能体排行榜通常以模型名与通过率排名，但模型外围的 harness/scaffold（下发工具、管理上下文、决定何时停止的运行框架）常被含糊处理。作者指出：仅当 harness 固定时，模型间比较才有效；一旦 harness 变化，性能与效率就混淆了模型效应与脚手架效应。本文把"仅固定模型、改变 harness"的受控研究系统化，提出"脚手架效应"，主张 harness 选择是编码 agent 评估中被忽视的隐藏变量，必须作为受控因子，且以"人本位"视角强调开发者真正关心的是每任务成本、完成时延与监督负担，而非仅模型名分数。

## 核心方法
在 Terminal-Bench Pro 的 50 个任务（跨 8 类原型 stratified 抽样、pytest 确定性评测）上，对两个近期强编码模型 Qwen 3.6 Plus 与 MiniMax M2.5，跨 3 个开源 harness（Goose 重度 IDE 预注入文件树；OpenCode 持久工具循环、无自动上下文预加载；OpenHands-SDK 微智能体架构、含子智能体委派与显式验证）共做 300 trials。统一项：原始任务说明、原生测试套件、Daytona 沙箱、900 秒墙钟上限、OpenRouter 网关、基本相同的 system prompt；不统一项（即研究对象）为各 harness 原生工具 API、自动上下文预加载与内部重试/子智能体逻辑（OpenCode 无 turn-budget 标志，仅受墙钟约束）。指标含 solved、turns、tokens_total（按调用方计费口径不跨供应商归一化）、no-action turns（既未改文件也未发命令，作为监督负担代理）、wall_seconds，以及 6 类失败分类（REASON/VERIFY/TIME/MAX_TURNS/HANG/ERROR）；通过率与每解决任务 token 数均报告 95% 配对任务 bootstrap 区间（B=10,000）。

## Performance 概括
配对通过率差异很小：同一模型内 harness 差异仅 2–8pp，跨模型同 harness 为 4–10pp，n=50 下多数配对差在 bootstrap 噪声内（最大 −8.0pp 的 CI 为 [−18.0, 0.0]）。但每解决任务 token 数（核心效率指标）排序两模型一致：Goose ≪ OpenHands-SDK < OpenCode，OpenCode 比 Goose 高约 40×，且非由 turn 数驱动（OpenCode 平均 21–27 turns，仅 Goose 的 ∼1.2×）；bootstrap 显示 Goose 上限（40–61K）远低于 OpenCode 下限（733K–1.01M），量级差稳健。OpenCode 平均 no-action turns 为 2.0–2.16，是 Goose（0.2–0.3）的约 10×，跨模型复现。失败指纹跨模型一致：Goose 为 REASON 主导（卡住即停、零 VERIFY），OpenHands-SDK 为 VERIFY+MAX_TURNS，OpenCode 为 TIME+idle spinning（零 VERIFY）；Pareto 图上 Goose 居前沿、OpenCode 被两模型双重支配。

## 论文价值与不足
价值在于用受控实验把"harness 是隐藏混淆变量"系统化为证据：成本侧不对称压倒性（harness 改 tokens/solved 40×，而模型升级仅 1.0–1.3×），且失败指纹跨模型复现证明其为 scaffold 属性而非模型属性；据此提出"人本位编码 agent 评估单位应为 harness–model pair"，并建议排行榜除通过率外报告 tokens/solved、no-action turns 与失败类别向量。不足：仅 2 模型、3 harness、50 任务，n=50 下许多通过率效应在 bootstrap 噪声内仅作描述性；Goose 经 Harbor 仅报 token 总量、无输入/输出拆分；turn-budget 不统一（OpenCode 仅墙钟）且"turn"本身为 harness 定义；结论不应外推至未评估的闭源商业产品或所有脚手架架构；OpenRouter/供应商默认参数可能随运行漂移。

## 工业界落地可行性评估
高度可行，且对编码 agent 选型与评测有直接指导意义。企业比较编程 agent 时必须固定或显式报告 harness，否则结论不可比；其失败指纹分类（REASON/VERIFY/TIME 等）可用于落地时区分失败源于 harness 还是模型，指导"谁的可信输出更可靠"的监督策略（如 Goose 卡住即诚实放弃、OpenCode 命中测试的解更可信）。论文同时提供 token/时延/监督负担等部署相关信号，适合编码 agent 供应商评估与上线前测试；配套已开源 harness 配置、原始 trial 日志与分析脚本（github.com/namanvats/scaffold-effects），便于复现与二次评估。
