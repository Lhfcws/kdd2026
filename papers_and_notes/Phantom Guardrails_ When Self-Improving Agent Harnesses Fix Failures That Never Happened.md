# Phantom Guardrails: When Self-Improving Agent Harnesses Fix Failures That Never Happened

## 背景与 Idea
自改进 agent harness 会从失败中优化脚手架（prompts、parsers、filters、validators、guardrails）。作者发现这类系统会"幻觉出从未发生的失败"——即 phantom guardrail（幻影护栏）：为一个可证明不会发生的失败类构建护栏。由于抑制失败（suppression）的奖励在"全合法"输入上早已满格，幻影护栏既不被奖励黑客也不被过度拒绝，却悄悄增加了脚手架的表面积与延迟。

## 核心方法
提出 Counterfactual Fabrication Lab（反事实虚构实验室）：一个确定性 micro-lab，正确动作预先已知为"do nothing"。植入一个"不存在规则"的护栏（castling 类失败在任务生成器中不可产生），仅向 proposer 展示合法 episode，并用 byte-exact oracle 检查每个被引用的违规是否真实。测试分两阶段：单发提案（proposer 只读自己输出，无接受反馈）与 add-only accept loop（接受后持久、不可移除）。变化池（pristine 全合法无特征 / fabrication 含 benign 表面规律 / congruent 真实违规）与三类机制控制（规则形状、规则集完备性、失败预设指令）。

## Performance 概括
在无特征合法池上，proposer 在 54/60 次返回空 harness（0/60 启用幻影护栏）；但加入 benign 表面规律后，15/60（0.25，95% CI [0.16,0.37]）启用不存在规则的护栏并引用 oracle 反驳的违规。虚构仅在三条件同时满足时发生（规则形状的模式 + 开放规则集 + 预设失败的指令），移除任一条件即归零（z=4.14）。在 add-only 接受循环中，幻影护栏一旦进入即永久留存，每轮进入率 q=0.050，从 1/60 轨迹累积到 round 4 的 11/60；而 warrant-aware 接受可全程排除它（0/60）。

## 论文价值与不足
价值在于揭示 suppression-rewarded 自改进 harness 中"虚构失败"这一全新失败模式，给出可审计（确定性 oracle、$0 成本）的测量工具，并指出三类杠杆：指令卫生（去掉失败预设）、规格完备（一句话声明规则集完整）、warrant-aware 接受（仅当能引用 oracle 确认的违规才接受护栏）。不足在于仅单一确定性 micro-lab、单一游戏类型先验；真实代码型 harness 与其他先验（如工具使用安全规则）的跨域验证是未来工作；效果虽统计显著（z=4.14）但绝对率约 0.25 且集中于个别 proposer（glm-5.1 11/12）。

## 工业界落地可行性评估
自改进 harness / agent 优化循环正进入生产（如自动 scaffold 搜索），本文警示"抑制失败"这一单一指标会掩盖幻影护栏带来的延迟/表面积/特异性税，对 agent 运维监控有现实警示意义；但需结合真实 harness 与领域先验验证后，再据此改造接受循环。
