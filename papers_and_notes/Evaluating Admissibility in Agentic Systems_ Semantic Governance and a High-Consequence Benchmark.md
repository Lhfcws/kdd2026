# Evaluating Admissibility in Agentic Systems: Semantic Governance and a High-Consequence Benchmark

## 背景与 Idea
可信 agent 的输出可能流畅、合理却不适合执行。在高风险工作流中，一个被提议的动作是否"可准入（admissible）"不应只看其似真性或任务准确率，而要看它是否有受治理的理由依据、权威支持与可回放的决策基础。论文指出现有可信/可靠 agent 研究通常停在"输出好不好/安不安全"，而没有回答"该动作在治理上是否被允许执行"，并把内部对齐声明转化为对外部可观测行动基础（证据、权威、策略兼容、可解释、可回放、可问责干预）的检查。其 idea 是把这一动作级控制问题定义为 admissibility（准入），并以资源-事件-主体（REA）事实到日记账分录（journal entry）的记账作为高后果压测域——它紧凑地暴露分类歧义、策略冲突与可审计要求，使"合理投影"与"可审查、可回放的决策基础"之间的差距可见。

## 核心方法
论文提出语义治理模型与执行规则骨架：Execute(a_t) 仅在 Justification 有效、Authority 批准、Replayable 三者同时满足时放行。核心对象包括 Admitted Facts → Governed Projections → Execution Intents，由 Justification Artifacts 与 Replay Bindings（内容寻址证据哈希、治理规则 id、权威上下文 id 的定钉决策包）把投影过渡到准入；控制边界分 Fact Admission、Projection Trust、Execution Admissibility 三层。基准包含 6 类场景（positive/correctable-negative/evidence-gap/policy-conflict/projection-rejection/replay），5 个 gold 标签 ALW/AWC/ESC/BLK/FPRR；治理路径（Semantier）在直接 LLM 投影前插入证据门、策略门、回放绑定与纠正/升级/阻断逻辑。评测维度含结果控制（记账正确性、unsafe-allow 率、纠正精度、正当阻断率）与解释质量（证据/理由可追溯、决策轨迹清晰度、回放一致性、决策前理由覆盖），并用冻结模型（openai:gpt-5.5，temperature 0）、固定 prompt hash、每案例 5 次种子试验保证可复现。

## Performance 概括
在冻结条件下对比 permissive direct、cautious、governed 三条路径。24 例合成包的整体记账正确率为 8/24（0.33，direct）、12/24（0.50，cautious）、24/24（1.00，governed），整体 unsafe-allow 率分别为 0.67、0.50、0.00。30 例公开来源派生基准（来自 Zenodo、PCAOB、SEC EDGAR XBRL 制品）上，unsafe-allow 计数为 14/30（direct）、7/30（cautious）、0/30（governed），对应记账正确性为 8/30、15/30、30/30；谨慎基线在 evidence-gap 类有改善，但在 correctable 误分类与 policy-conflict 类仍不如治理路径的干预分离清晰。外部独立双人评分对 30 例结果标签达成 30/30 一致（Cohen's κ=1.00），次级字段一致性较低（纠正充分性与解释依据 76.7%）。结果应读作"定钉条件下治理协议的受控证据"，而非跨会计或 agent 决策领域的泛化优势。

## 论文价值与不足
价值在于把"是否允许执行"从模糊的对齐目标变成可度量、可回放、可审计的评估对象，并用冻结制品、外部双人裁定、组件归因表把治理路径与直接基线在干预行为上做了清晰区分，方法学上对高风险 agent 执行控制有直接参考意义。不足在于：24 例合成包为作者自造，存在把期望答案编码进基准的循环风险；30 例公开来源包的案例包、JE 金标与治理标签仍属作者标注，外部制品只提供字段与缺陷模式；direct/cautious 基线是有意收窄的比较族而非完整基线套件，故属受控干预研究强、前沿模型性能声明弱；每门控贡献的消融（hold-out 消融）尚未完成；生命周期回归监控虽为协议设计但本次未经验证。

## 工业界落地可行性评估
该框架对会计、审计、金融等高后果、强监管的动作级执行控制有明确落地场景：其四门控架构（结构 REA 校验、分层证据门、策略与决议门、回放绑定校验）可直接嵌入企业 agent 的"allow/correct/escalate/block"决策流，与模型卡/数据表的可追溯理念一致，适合作为上线前的治理闸门与人机协同升级点。但落地需注意：治理质量仍依赖策略质量本身，回放问责不能替代人工策略设计；治理会引入审查与运营开销，在部分环境可能不可接受；当前仅在小规模冻结切片验证，工业部署应扩展真实工作流轨迹并补齐逐门控消融与生命周期回归监控后，再将其作为生产治理层而非性能认证依据。
