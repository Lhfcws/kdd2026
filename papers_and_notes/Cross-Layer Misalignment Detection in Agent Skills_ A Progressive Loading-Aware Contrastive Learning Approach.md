# Cross-Layer Misalignment Detection in Agent Skills: A Progressive Loading-Aware Contrastive Learning Approach

## 背景与 Idea
Agent Skill（如工具/API 使用技能）常在多层表示——语义描述、参数 schema、执行轨迹——之间出现"跨层错位"：技能名义上对齐，但实际行为已偏离。这种错位会让 skill 库在检索/调用时产生不可信行为。作者提出 PL-HCL（Progressive Loading-Aware Hierarchical Contrastive Learning）来检测这种跨层错位。

## 核心方法
构建包含 264,937 个归一化"包"（skill 不同层表示单元）的训练语料，并构造 Challenge Set 共 1,444 个样本（1,150 aligned，294 misaligned）。PL-HCL 采用渐进式（progressive）、加载感知（loading-aware）、分层对比学习，将技能各层表示对齐到统一空间，使对齐样本表示相近、错位样本表示远离。基础模型用 Llama-3.1-8B，并与安全模型 Foundation-Sec-8B 对比（后者采用 two-shot 设置）。

## Performance 概括
在 Llama-3.1-8B 上，CPT+PL-HCL 达到 Macro-F1 0.872；在 Foundation-Sec-8B 上达到 0.889（two-shot）。基础（base）模型 Macro-F1 约 0.52，显示 PL-HCL 显著提升了跨层错位检测能力。

## 论文价值与不足
价值在于首次系统定义并检测 agent skill 的跨层错位，提供了一个可扩展的对比学习框架与大规模 Challenge Set，对 skill 库的质量与安全治理有意义。不足在于：主要基于归一化包与抽取/合成数据，真实生产 skill 库的噪声与分布偏移未充分验证；Foundation-Sec-8B 在 two-shot 下的提升是否来自其安全预训练需进一步分析；方法的误报/漏报在生产阈值下的代价未讨论。

## 工业界落地可行性评估
Agent skill 市场与平台（如工具调用技能库、企业内部 skill 目录）需要自动化质量与一致性审计，本文方法可作为 skill 上架前的跨层错位检测组件，落地可行性中等偏高；但在真实噪声 skill 库上验证前不应直接用于硬性拦截。
