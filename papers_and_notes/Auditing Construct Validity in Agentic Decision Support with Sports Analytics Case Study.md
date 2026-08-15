# Auditing Construct Validity in Agentic Decision Support with Sports Analytics Case Study

## 背景与 Idea
多数 agent benchmark 只问"任务是否完成"：WebArena 检查网页流程是否走完、SWE-bench 检查代码是否通过测试、𝜏-bench 给工具-代理-用户交互打分。这些系统都有可验证的完成信号（oracle）。但一类新兴系统产出的是给人类决策者的"分析性断言"（analytical claim）——临床决策支持笔记、组合风险归因、给足球主教练的战术分析报告。这类交付物对世界做出判断，例如"我们对右半区出球型对手的高位压迫会崩溃"，而消费方必须同时信任论断的"实质"与"方法"。这里没有留出测试集、也没有 ground-truth 标签，正确问题更接近"领域专家是否会为这个论断辩护"。

论文把"构念效度（construct validity）"引入该场景。构念效度源自心理测量学：Cronbach 与 Meehl 提出、Messick 扩展，后来 Jacobs 与 Wallach 把它用于机器学习公平性（测量与公平）。它衡量"一个测量工具是否真正测到了它声称要测的那个构念"。近期 Brookings–CMU–NIST 研讨会综述（Tabassi 等）明确建议把测量科学概念（构念效度、内容效度、预测效度）适配到 agentic AI，而非临时拼凑评估方法，但止步于呼吁、未做端到端操作化。作者提出 TRACE——一个四威胁审计框架，把构念效度落地到 agent 产出的分析性论断，每个威胁配一个可度量审计工具，并在足球战术分析上实例化为诊断性 benchmark PressureBench。足球分析被选作压力测试有其用意：pressure、compactness、defensive effectiveness 等概念没有 universally accepted 的操作定义，构念效度问题格外突出。需要区分的是：Phiri 用八条公理形式化了多智能体可审计性的日志底层；Deloitte AI Assurance 白皮书盘点了两类工业做法（组件级 Arize/Orq/IBM、断言级 Apple/AWS/NEC，以及 Galileo/LangSmith/Arize Phoenix 等监控平台）——但它们都审计组件健康与轨迹合规，没有一篇审计"论断本身的方法论效度"。Zimmer 等的生产者侧"十条戒律"（如"绝不伪造引用"）防止 agent 内部出错；而 TRACE 审计"已产出的东西"，正是这些生产者框架缺失的互补层。论文贡献是"操作化"，概念本身并非新东西。

从更宽的评估视角看，四个威胁构成一条有序依赖链：先要有论断（claim）、才能有判定（verdict）、判定需在多次运行（run）间一致、且跨模型版本（version）不退化——这与 Shukla 等提出的五轴分类（含目标漂移、减害等）不同，TRACE 是依赖链而非需要权衡平衡的若干轴。论文把生产者侧研究作为背景也很关键：Lu 等的端到端科学产出管线在 Nature 发表、且通过 ICLR 2025 workshop 同行评审，却仍在自身局限里列出"缺乏深层方法论严谨性"与幻觉；Zimmer 等的框架用十条嵌入提示的戒律（如绝不伪造引用）防止 agent 内部出错。这说明"agent 能产出什么"已有较多工作，而"agent 已产出的论断在方法论上是否有效"才是被忽略的互补环节，正是 TRACE 所审计的。足球分析被选作实例化还有一层原因：pressure、compactness、defensive effectiveness 等概念没有 universally accepted 的操作定义，构念"是否被一致地操作化"问题格外突出，使该领域成为检验构念效度的天然压力测试。

## 核心方法
TRACE 将一个已部署系统的交付物 𝐷 拆成若干"分析性论断"𝑐𝑖（如一条带构造与验证流程的指标提案），把单条论断作为最小评估单元——注意原子单元是论断而非整个交付物或任务完成度。它识别四个针对 agent 论断的构念效度威胁，并各配一个可度量仪器：

T1 构念误设（construct misspecification）：agent 操作了错误目标。PressureBench 用六种在已发表足球分析文献中锚定的方法论反模式来实例化：E1 把 provider 标签当监督目标、E2 缺失防守区块分层（高/中/低）、E3 仅用回合内验证（只算立即恢复、忽略延迟/威胁减少/被迫回传）、E4 仅用案例研究作证、E5 缺失归一化（原始计数未控制控球或推进量）、E6 缺失阶段条件化（跨结构不同的对手阶段聚合）。对应仪器 APCR（反模式捕获率，按 ground-truth 反模式是否被检入集合算，配合 hard distractor 上的 FP 率共同读，避免 carpet-flagging 刷到 1.0）。

T2 判定不可靠（verdict unreliability）：称职评审对同一论断给出不同判定，单一 judge 不够。仪器 IAJA（多 judge 一致性）报告两两 Cohen's κ、面板 Fleiss' κ，并在有保留子集时对比每 judge 相对两位专家共识的准确率；κ≥0.61 视为 substantial（Landis & Koch 标准）。关键在于用专家对比防止"judge 们在错误答案上达成一致"。

T3 跨运行随机不稳定（stochastic instability）：同一 brief 多次重跑若给出不同构造，构念效度评估就不可靠。仪器 SRI（科学可复现指数）对每个 brief 重跑 𝑁=5 次，组合三个两两 Jaccard：formula-component（指标词汇）、cited-method（命名方法）、verdict-exact-match（审计判定完全一致的比例）；verdict-match < 0.5 即标记"单次 producer 调用不安全"。

T4 模型升级回归（model-evolution regression）：昨天能用的审计管线，上游 LLM 升级后可能失效。仪器 MERS（模型演化回归分）把冻结子集在每个模型版本下重放，算每反模式的 ΔF1；用 per-AP（而非聚合）阈值告警，因为各反模式回归幅度差异极大。

架构上有一个追加式（append-only）统一事件日志 events.jsonl，记录每次 LLM 调用的 model id、response id、token 数、prompt 版本、完整原始响应，以及 judge 判定、rule 触发、论断生成与实验标记；prompt 与模型版本标签使任意历史审计状态可重放——这是 MERS 可复现的承重特性。日志覆盖 Phiri 八公理中的 Coverage、Temporal Coherence、Accessibility，但加密级防篡改不在范围内（受监管部署的自然加固层）。PressureBench 共 38 条防守压迫分析提案：24 条种子化植入六种反模式（E1–E6 各 4 条，均锚定已发表足球分析文献，其中 Forcher 等在自己讨论里承认"本研究弱点"仍照发队均压迫统计、Morgan 自承"可能循环关系"仍发未归一化排名），6 条有效基线，4 条 hard distractor（表面像反模式实则合理，用于测 per-surface FP），4 条由 Opus 4.7 从无偏分析师场景提示生成（不提 TRACE/评估/反模式分类）以破除自评分担忧。工业管线把 brief 经过 producer→四审计仪器→事件日志→审计标注的战术分析报告（固定六段：执行摘要、per-block E2 审计、per-phase E6 审计、含 E5 警告的球员附录、建议、显式非断言），每周在冻结子集上跑 MERS 重放。

六个反模式各自锚定一篇确实犯该错的已发表足球分析文献，且两条锚定尤其强：Forcher 等在自身讨论中明言缺少区块分层是"本研究弱点"却仍照发队均压迫统计；Morgan 报告其排名指标与控球份额相关性 r=0.86、自承"可能循环关系"却仍发未归一化联赛排名。这种文献锚定正是为防御评审者"种子错误是稻草人"的质疑——错误来自真实发表物。四个 hard distractor 也各有对应反模式的表面信号以便测 per-surface 误报率：一个仅把 provider 标签当健全性检查（E1 表面）、一个在系统分层分析旁附案例片段（E4 表面）、一个给每区块联赛排名却明确拒绝跨区块比较（E2 表面）、一个给按阶段加权球队评分且含每阶段子分（E6 表面）。

多 judge 校准方面，三 judge 面板由 Sonnet 4.6、Opus 4.7、Haiku 4.5 组成：在 E1 上达 Fleiss κ=0.89（substantial），其余中等，E3 最低仅 0.44；三位 judge 严格 APCR 都饱和到 1.00，但 Haiku（最小模型）在有效锚上 FP 最低（0.17）、hard distractor 上完美 0.00，Opus 在 distractor 上 FP 最差（0.75）。两位同方法论训练的人类专家对 16 条盲法子集给判定：相对 AND 共识的召回 0.87–0.93、相对 OR 的精度仅 0.30–0.33（judge 普遍过度标记），无任何单一 judge 跨全部反模式占优（Haiku 在 E1/E6 最佳、Sonnet 在 E3/E5、Opus 在 E4），这支撑了按反模式选层的部署建议。

## Performance 概括
四个部署相关发现：(i) 能力层级 ≠ 精度代理——最小模型 Haiku 4.5 在有效锚点上 FP=0.17（95% CI [0.00,0.50]），最大 Opus 4.7 达 0.67 [0.33,1.00]，hard distractor 上 0.00 vs 0.75（CI 不重叠）；(ii) 单次 producer 调用不安全——N=5 重跑中仅 17% 得到相同审计 verdict，且换 Sonnet 4.6 复现同一 0.17；(iii) 每反模式回归差异大——E1 ΔF1=0（evergreen），E3 达 0.35（regression-prone）；(iv) 对抗式 producer 压力测试（避用触发词）中审计管线仍捕获 10/12=83%。简单 rule 层在 6 个反模式中 4 个上胜过 LLM judge（如 E1 F1=0.89）；整轮评估 276 次 LLM 调用仅花费 $10.90。

## 论文价值与不足
价值在于把测量科学中的构念效度首次端到端操作化为"框架 + benchmark + 多 judge 校准 + 重跑协议 + 模型演化监控"，并公开全部数据/脚本/日志（https://github.com/Peggy4444/PressureBench-TRACE）。识别了"单次 producer 调用不安全""小模型 FP 更低""OR 组合 disjunctive 管线在四反模式上 F1 反而最低（因 recall 已饱和，OR 只增 FP）"等反直觉结论，并给出五条部署建议：R1 按精度画像而非能力层级选 judge（Haiku 比 Opus 便宜约 15× 且精度更高）；R2 按反模式选层、不要 OR 组合（E1/E4/E5/E6 用规则，E2/E3 用 LLM judge）；R3 重跑 𝑁≥5 并多数投票聚合；R4 按反模式分别告警（全局阈值要么漏 E3 要么误报 E1）；R5 新领域靠三件套落地（文献锚定分类、PressureBench 形态 benchmark、produce(spec) 实现），TRACE 代码与日志 schema 不变、只重建 benchmark。失败分析把八类 LLM 失败模式映射到 TRACE：三类被捕获（数据泄漏→E1、缺失归一化→E5、缺失区块/阶段池化→E2/E6），两类部分捕获（过度自信论断、无效统计推断），三类需扩展本体（引用伪造→文献核查、战术幻觉与伪造领域概念→本体接地）。不足在于：仅足球一个领域、仅 Anthropic Claude 一家模型，跨域跨厂商泛化未验证；每反模式 n=4，点估计尚属初步（v1.2 计划 n≥8）；原子单元是单条论断而非完整交付物；引用伪造、战术幻觉等失败模式需扩展本体接地与文献核查才能覆盖；人类 ground-truth 仅 16 条提案，跨评分者研究样本偏小。论文把当前 agentic AI 评估比作"莱特兄弟阶段"，TRACE 是其中之一而非终点。

八类失败模式被显式映射到 TRACE：三类被捕获（数据泄漏→E1、缺失归一化→E5、缺失区块/阶段池化→E2/E6），两类部分捕获（过度自信论断经 E2/E5、无效统计推断与 E3 重叠），三类需扩展本体（引用伪造→文献核查、战术幻觉与伪造领域概念→本体接地）。最说明问题的具体案例是数据泄漏：SRI 中 20 次 Opus 重跑有 12 次基于 SkillCorner provider 标签训练，PB-019（"adapt a published method"）直接复刻了已发表的 E1 缺陷而非质疑它——规则与 judge 都能捕获。引用伪造在生产者侧已由 Zimmer 等的第三条戒律覆盖，审计侧的自然扩展是接 Semantic-Scholar / OpenAlex 做核查。自评分防御由四步构成：文献锚定反模式、LLM 无偏生成子集（PB-017–PB-020）、专家校验子集、对抗式生产者压力测试（10/12=83%），单步在本研究样本量下都不具决定性，合起来才转移举证责任。

## 工业界落地可行性评估
论文明确给出"工业管线"形态（producer → 四审计工具 → 审计标注报告 → 主教练），主张与 Deloitte 等现有监控平台互补而非竞争——Galileo/LangSmith/Arize Phoenix 关注组件健康与轨迹合规，TRACE 关注论断本身的构念效度，生产部署应两者并用、受监管时以 Phiri 日志底层托底。落地集成契约只是一个 produce(spec) 方法，框架对 producer 与 detector 皆不可知，直接面向足球战术分析等工业决策支持场景，落地可行性高。当前范围窄，真实落地需跨域验证与监管级（可篡改证据的加密加固）日志；但作者已给出跨领域扩展路径（临床决策支持、金融风险归因均共享"分析性论断"部署模式，可从其方法论文献构造类似种子错误 benchmark），短期内即可作为 CI/CD 之外的"判定质量"监控层接入。

落地集成的契约极轻：只要实现一个 produce(spec) 方法，TRACE 的代码、事件日志 schema 与四个支柱仪器都可原样迁移，仅 benchmark 需按新领域重建。受监管部署的自然加固层是给统一事件日志加加密级防篡改（Phiri 八公理中当前未覆盖的部分），使每次 LLM 调用、每个判定都可被独立审计与重放。成本侧也利于落地：整轮 276 次 LLM 调用仅 $10.90，纯 Haiku judge 跑一遍 PressureBench 可压到 $0.20 以下，意味着把 TRACE 作为常规监控轮次运行的经济门槛极低。对任意采用者，从足球迁移到新领域的三件套是：一份文献锚定的反模式分类、一个 PressureBench 形态的 benchmark（有效锚 + 反模式种子 + hard distractor）、一个 produce(spec) 实现。
