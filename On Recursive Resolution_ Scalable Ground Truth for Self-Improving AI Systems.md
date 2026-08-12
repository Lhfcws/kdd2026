# On Recursive Resolution: Scalable Ground Truth for Self-Improving AI Systems

## 背景与 Idea
复合检索增强问答（compound RAG）系统由检索、排序、生成、安全等多个组件交互产生涌现式对话行为，传统人工评测难以扩展，而 LLM-as-judge 又面临"校准需要 ground truth 标签、但生成标签本身正是自动化要消除的事"这一自举悖论（bootstrapping paradox）。论文要解决的核心问题是：在有限标注预算下为复合 AI 系统建立可靠 ground truth，并据此做统计严谨的决策。其 idea 是把"LLM 与人工一致"当作免费信号，仅在二者分歧时升级到第二名标注者，从而以单标注者成本达到多标注者可靠性；再用分层 Wilson 区间把噪声化二元判断转为带置信界的评估结果；最终将评估阈值与修复效果耦合，形成"系统自我改进自身检测标准"的闭环架构（recursive resolution）。

## 核心方法
方法由三部分构成。其一是两轮迭代共识合成（iterative consensus synthesis）：Round 1 由 LLM judge 与人工标注者 H1 对每个对话在 13 个质量指标上独立判分，完全一致（13.2% 概率）的对话直接把一致结果作为 ground truth，分歧的 86.8% 才升级到 Round 2，由 H2 独立判分后按三人多数投票定标。其二是分层 Wilson 区间决策框架：对每个指标用 Wilson 下界 L_i 与基线失败率 θ_i 比较，BLOCKER 任一回归即 Fail，≥3 个 P0 回归即 Fail，单个 P0 回归为 Investigate，仅 P1 问题则 Pass。其三是五层闭环架构：Evaluation 层（LLM judge 评 13 指标，分 BLOCKER/P0/P1 三优先级）→ Root Cause 引擎（语义聚类并归为 6 类根因）→ Remedy 引擎（提修复方案，需人工批准）→ Rollout 管理器（shadow 部署 + 安全/质量/时延门禁）→ Continuous Learning 层（按修复效果回写 Layer 1 的检测阈值与规则），从而让评估结论反哺修复、修复效果再精炼评估标准。

## Performance 概括
在复合 RAG 系统、13 个质量指标、3 个 locale 测试集上验证：合成对话生成器产出 1,946 段多轮对话（平均 7.53 轮），经质检后 1,397 段进入评测语料，每个新增 locale 再生成 555 段。LLM judge 在 BLOCKER 指标上与共识 ground truth 对齐达 99%+（如 PII 99.9%、Offensive 100%），主观 P0 指标对齐 73.6%–87.6%，与单个人工标注者相当；两轮机制将标注工时从基线外推的 7,672 小时降至 481 小时（15.95× 缩减），评测覆盖轮次从 3,600 提升到 14,653（307% 提升）。案例一中针对第二 locale 内容格式发散的修复经 200 段 shadow 验证使 Knowledgeable 从 58.3% 升至 79.1%（+20.8pp），全量测试集失败对话降 42%，Layer 5 还把该模式泛化到后续 locale 提前规避回归；案例二在第三 locale 用 Wilson 区间识别出 2 个 P0 失败，触发 Pass-with-Investigation。

## 论文价值与不足
价值在于用简洁的"一致即免费信号 + 分歧才升级"机制打破了 ground truth 自举悖论，并以 Wilson 区间把噪声判断转为可统计决策、以递归闭环把评估与修复打通，在 Amazon 真实 RAG 场景上同时实现了成本下降、覆盖提升与完整改进闭环，对生产级连续监控很有说服力。不足在于：ground truth 质量仍依赖标注者领域专长，Wilson 框架假设指标独立但实际存在重叠（第三 locale 中 Unhelpful 与 Unknowledgeable 失败有 50% 重叠）；Layer 3 修复仍需人工批准，完全自主修复是未来工作；评测聚焦英文，多语言扩展未验证；且当前只能判断问题存在、无法定位到具体组件（retriever/ranker/generator）。

## 工业界落地可行性评估
该架构面向企业级复合 AI 系统（尤其 RAG 客服/助手机器人）具有很强的落地可行性：迭代共识合成可直接削减大规模标注成本，分层 Wilson 区间提供了可审计、可复现的发布门禁，五层闭环也可嵌入现有 MLOps/影子发布流程；案例已来自 Amazon 生产实践，说明其与真实工程栈兼容。落地注意点包括：需保留人工在修复批准与根因标注上的把关、需为指标间相关性做修正或分层聚合、并把组件级归因与多语言/语音模态扩展作为后续建设项，整体是一项高可行性且收益明确的生产评测方案。
