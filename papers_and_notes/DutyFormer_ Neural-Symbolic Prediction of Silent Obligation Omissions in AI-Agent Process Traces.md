# DutyFormer: Neural-Symbolic Prediction of Silent Obligation Omissions in AI-Agent Process Traces

## 背景与 Idea
在 AI-agent 的流程（process traces）中，常发生"沉默义务遗漏"（silent obligation omission）：某个本应执行的义务被悄悄跳过，既没有显式报错也不会立即暴露，却可能累积成合规或流程风险。论文关注如何自动检测这类"该做却没做"的遗漏。

## 核心方法
DutyFormer 是一个 Neural-Symbolic 模型，核心包含 SAB（symbolic-aware block，符号感知模块）与 conformance vector（合规向量），把语义义务与符号约束融合进表示。模型用 conformal prediction 给出带覆盖保证的预测，并在 LODO（leave-one-domain-out）与 TSuC 设定上评估跨域泛化，同时用 τ-bench 做外部验证。

## Performance 概括
在 omission miss rate（OMR）上，DutyFormer 为 0.95×10⁻⁴，显著优于 RoBERTa-PM 的 9.70×10⁻⁴（约 10 倍优）。消融显示：移除 SAB 使 OMR 翻倍（1.91）；移除 conformance vector 使 OMR 升约 57 倍，说明两个组件都关键。conformal coverage 达到 90%；在 LODO 下 TSuC 降至 0.7791，跨域有一定下降。

## 论文价值与不足
价值在于把语义义务与符号约束结合，给出可校准、带覆盖保证的遗漏检测，比纯神经基线更可靠。不足在于 LODO 跨域性能下降明显，提示对未见流程域仍较脆弱；虽已用 τ-bench 做外部验证，但真实流程日志的覆盖域仍有限。

## 工业界落地可行性评估
可落地于合规审查、流程审计与 RPA/agent 监控系统中，对"该做未做"类风险有实际预警价值。其 conformal 覆盖保证也便于在监管场景做风险预算，落地路径清晰。
