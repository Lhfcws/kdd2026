# Numerical Format as a Post-Deployment Risk Surface: A Joint Capability–Safety Audit of FP4 Vision–Language Models

## 背景与 Idea
生产环境中 VLM 的部署产物并不稳定，常因推理成本优化而静默切换数值格式（FP16 ↔ MXFP4 ↔ NVFP4），这类 post-training quantization 由工具链（如 TensorRT Model Optimizer）驱动，而非安全评审。论文主张：对 agentic/多模态系统，"数值格式"本身就是部署后风险面的一部分——一个运行时侧的量化变更可以改变化害请求被拒绝、遵从还是升级路由，却不会改变权重哈希、系统提示或工具面，因此对 weight-hash / prompt-version 监控完全不可见。这是首个针对生产级 VLM 上 FP4 推理格式的能力–安全联合审计。

## 核心方法
固定架构与训练流程，仅变动推理数值格式（FP16 基线、MXFP4、NVFP4），在 5 个开源 VLM 上评测：Qwen2.5-VL-7B、InternVL3-8B、LLaVA-OneVision-7B、Idefics3-8B-Llama3，以及作为 Idefics3 负向对照的 Idefics2-8B（共享 SigLIP 视觉塔但 Mistral-7B 解码器）。能力轴用 MMBench-V11（n=1292，四选一字母准确率，贪心解码）；安全轴用 MM-SafetyBench 全部 13 类、含 text-only 与 SDTYPO 图像提示两种模态，每场景每模态 n∈{8,16,24}（对应每 cell 208/416/624 条，两个失败模式 cell 在 n=48 即 1248 条做更紧 CI）。判定用双 judge AND 共识：14 模式高召回正则拒绝短语 + 指令微调 LLM judge（默认 Qwen2.5-3B-Instruct，二值强制、3 个 few-shot），主安全指标为 refusal_decided（两 judge 一致同意 REFUSE/COMPLY 的拒绝率），用配对 bootstrap（B=2000）给 95% CI。

## Performance 概括
任一 PTQ cell 都没能在两轴同时优于 FP16，即"无 FP4 免费午餐"。MXFP4 产生三种架构性迥异的失败模式：① 欠拒绝双降级——InternVL3 能力 −0.143、安全 −0.105（n=1248）；② 过度拒绝 + 能力崩溃——Idefics3 能力暴跌 −0.263（全组最差），但拒绝行为显著正向 +0.178（n=1248），联合标签分布倒向拒绝式短语；③ 仅能力损失——LLaVA-OneVision（−0.071）与 Idefics2（−0.065）能力下滑但拒绝行为近似零。Qwen2.5-VL 近干净（能力 −0.015），NVFP4 整体更温和（在 InternVL3 上回收 11.4pp 能力、5.3pp 安全；Idefics3 上回收 19.8pp 且过度拒绝消失）。模态不对称被 MXFP4 放大：如 illicit activity 的 text-vs-image 拒绝差在 Qwen 上 +0.54→+0.88、InternVL3 上 +0.33→+0.79。样本量实验显示 n=8 pilot 夸大了 MXFP4 足迹（Qwen/LLaVA 的边缘效应在 n=24 时消失），仅 InternVL3 效应样本稳健。组件消融定位 Idefics3 过度拒绝在 LM 侧（mxfp4_lm Δ+0.120）、能力崩溃在视觉侧（mxfp4_vision Δacc −0.200），且独立 Llama3.2-11B-Vision 检查否定了"通用 Llama3"假设。3B→7B judge 升级下符号与最坏排序均保持。

## 论文价值与不足
价值在于把"量化格式切换"确立为 agentic VLM 部署治理中一个被忽视的评测轴，并证明能力–安全必须联合审计：Idefics3 案例说明只盯 MMBench 式准确率监控会因"错误原因"（能力崩溃）报警，却漏掉拒绝行为同时漂移；给出可迁移的最小可用 LLM-as-judge 审计模板（高召回启发式 + LLM 判官 + 公布容量消融防 judge 主干伪影）与"功率感知回归测试"教训。不足：仅用单一安全基准 MM-SafetyBench（缺 HarmBench/MM-Vet 第二套以排除套件特异性）；Llama Guard 3 受 gated 未纳入，3B/7B 检查只覆盖 judge 容量轴而非家族轴；InternVL3 失败的机制尚无完整因果解释（缺并行 LM/vision 消融）；纯描述性、无缓解实验（如 SmoothQuant 校准能否挽回）；效果为架构相关而非量化理论可单凭激活离群预测。

## 工业界落地可行性评估
对实际部署 VLM agent 的团队，该文有直接治理价值：应将推理数值格式与模型权重、提示模板同等对待，纳入版本管理与告警（格式 swap 可让拒绝率无声移动 10–18pp）；上线前/格式变更后必须跑"能力+安全"联合回归而非只看任务准确率，并设置架构感知阈值。其三点实战建议尤其适合落地：① 把数值格式作为与权重同级的监控面；② 小 n 试点（如 n=8）只够分诊、不足以声称"格式 F 对模型 M 安全中性"，生产级回归测试必须在安全轴配足统计功效；③ 采用双 judge AND 共识 + 容量消融的 judge 审计模式以防判官伪影。局限在于结论基于开源 VLM 与单安全基准，且未给出缓解方案，工业界采纳时还需结合自身模型族与多套对抗基准验证，并补充量化校准等缓解实验。
