# Comparative Analysis of Agent Evaluation Frameworks: Stability, Detection, and Discovery in Enterprise Analytics Agents

## 背景与 Idea
企业在生产环境部署 LLM 分析 agent（生成 SQL、调用工具、综合答案、面向敏感业务数据）时，错误输出有直接的运营后果，且常逃过人工 review。论文举了一个真实案例：一个系统性 text-to-SQL bug 静默地对整类查询返回零行，因为生成的 SQL 语法有效，靠人工抽查始终没发现。这类"语法正确但语义错误"的静默失败正是本研究焦点——作者指出大多数团队依赖的临时抽查漏掉了 72 个真实失败中的 9 个（12.5%）。

研究缺口在于：三种评估范式已经出现（LLM-as-a-Judge、确定性启发式、混合），但从未有人系统比较它们在同一组 agent 上、跨"稳定性（stability）/已知失败检测（detection）/未知风险发现（discovery）"三维度时的表现。论文从七个候选框架中定性筛选三个，分别代表三种不同评估哲学，在两个零售供应链预测分析 agent 上做三阶段比较，回答三个研究问题：评估稳定性（RQ1）、已知失败检测（RQ2，且答案是否依赖失败类型）、未知风险发现（RQ3）。

贯穿全文的统一主题是"信息获取层级（information access hierarchy）"决定评估质量：评估器能拿到的信息量决定其稳定性与检测能力。三个框架对应三个信息层级——Agenta 只看字符串表示（确定性启发式、零 LLM 成本），PromptFoo 看文本输出加注入上下文串（text-only LLM judge），Strands 看完整的 OpenTelemetry 执行轨迹（trace-based LLM judge）。本研究是首个在同一组 agent 上定量比较"功能评估"与"安全评估"互补性、而非把二者当可替代的工作。

论文的"客户影响声明"把问题落到运营后果：一个症状学上有效但语义错误的 SQL 会让商业智能系统的下游决策质量静默退化，而未被检测的 schema 泄漏漏洞会把内部数据库结构暴露给对抗用户。当前大多数团队依赖临时抽查，而本研究显示这种抽查漏掉 72 个 ground-truth 失败中的 9 个（12.5%），且漏掉的都是语法正确、人工目检看不出的语义错误——这正说明需要自动化评估策略。三个信息层级并非作者空想，而是对应评估器工程上能拿到的真实信息量：Agenta 只能拿到字符串（如 SQL 文本、正则匹配），PromptFoo 拿到 agent 回复加注入上下文串，Strands 拿到 OpenTelemetry 完整执行轨迹（含每步工具调用与中间结果）。论文还指出代理系统随机性会跨多步与多工具累积，因此稳定性测量本身就很关键。

研究缺口的提法很具体：三种评估范式（LLM-as-a-Judge、确定性启发式、混合）虽已出现，却从没人系统比较它们在同一组 agent 上、跨稳定性/检测/发现三维度时的表现。作者点出，既有 LLM-as-a-Judge 比较大多聚焦"评判方法的变化"（换不同 judge prompt 或评分策略），而非"评判者能拿到的信息量"这一更根本的变量——本文正是比较全执行轨迹、纯文本、纯字符串这三个信息层级下的评估质量。这也解释了为何选 frozen execution traces：所有框架都评同一批冻结轨迹，才能公平归因于信息层级而非轨迹本身差异。论文从七个候选框架中定性筛出三个，标准是它们代表三种不同评估哲学且都能映射到 Table 1 的五个评估器需求（Intent、Groundedness、Tool correctness、Accuracy、SQL correctness），从而把"任意 agent 都适用的固定结构"作为降低采用门槛的关键设计。

## 核心方法
被评估的两个 agent 来自零售供应链预测系统：Agent A 做品类级需求预测（9 个工具、数据量中等），Agent B 做产品级入库前置期预测（9 个工具含代码解释器、大规模生产表），二者产生不同 SQL 错误模式——一个结构上可检测、一个语义上才可检测。所有阶段都用冻结执行轨迹（frozen execution traces）以保证跨框架公平比较，研究明确说明这是受控研究设置、不是真实生产部署。

三个框架各自的五个评估器需求映射在 Table 1（这是适用于任意 agent 的固定结构）：Strands 基于完整 trace，覆盖 FaithfulnessEval、ResponseRelevanceEval、OutputEval、TrajectoryEval、OutputEval+SQL from trace（Intent/Groundedness/Tool correctness/Accuracy/SQL correctness 五维）；PromptFoo 基于文本，用 FaithfulnessEval(text)、g-eval(goal/rubric)、llm-rubric(tool names)、llm-rubric+SQL injection、Token/Levenshtein/Set/Regex 等；Agenta 纯确定性启发式（Token overlap、SequenceMatcher、Set comparison、Regex matching 等，零 LLM 成本）。

Phase 1 稳定性（RQ1）：每 agent 20 个领域专家准备的用例 × 3 个 judge 模型（Claude Sonnet 4.6、Claude Haiku、Qwen3 32B）× 5 次重复（Agenta 仅 5 次）；测两个量——决策一致性（decision consistency：重复是否给出同一 pass/fail 判定）与平均 within-case 标准差（稳定性阈值 ≤0.15）。阈值有操作化动机：在 0–1 标度、pass/fail 边界 0.5 处，within-case std 0.15 意味着边界附近的分数有不可忽略的概率跨过阈值翻转判定。论文特意采用 within-case 标准差（临床信度研究的标准测量误差指标）而非 ICC，因为 ICC 在表现良好的 agent 上会因天花板效应失效。

Phase 2 已知风险检测（RQ2）：每 agent 36 个用例（在 Phase 1 的 20 例上加 16 例），跨 6 类失败、沿 5 个维度独立标注以消除交叉污染（标注由每 agent 的主领域专家完成，歧义用例再由两名成员交叉复核）。五个维度是 SQL correctness、Tool correctness、Accuracy、Intent、Groundedness。冻结轨迹由三框架各评 3 次、取中位分、阈值 >0.5；judge 统一用 Phase 1 中稳定性与评估深度平衡最好的 Claude Sonnet 4.6。

Phase 3 未知风险发现（RQ3）：在 Phase 2 基线上加框架专属生成——Strands ExperimentGenerator 每 agent 20 个功能用例（带去重），PromptFoo 对抗红队每 agent 42–48 例；同一领域专家把每次被标记的失败分为 novel（开发 agent 时未知的模式）/known/假阳性，judge 仍为 Sonnet 4.6。三阶段共 5,605 + 2,415 + 3,680 = 11,800 次评估。

Phase 1 还揭示了一个对部署很实用的点：在三个 judge 模型中，Claude Haiku 最稳定（平均 within-case std 0.029），Sonnet 4.6 次之（0.035），Qwen3 32B 最高（0.072）；唯一不稳定配置是 Strands 上的 Qwen3/style（std 0.207、决策一致性仅 20%），但同一模型在 PromptFoo 中稳定（0.011），说明稳定性是（框架, 模型）配对属性而非模型单独属性——团队在某个 agent 上验证过的配置可直接信任于其他 agent。Phase 2 中两个被人工误标的语义错误案例很能说明 LLM judge 的价值：agent_a 生成品类级查询含 `partition_level='top'`，语法有效且返回正确结果，故人工标"通过"，但拿到执行轨迹全文与正确分区模式 rubric 的 SQL 评估器给 0.20–0.30（失败），人工复核后确认是分区错误；agent_b 计算入库前置期校准指标时误读了校准系数方向，在数据显示系统性低估时却报"校准良好"，不熟悉该指标解释的人工审阅者接受了 agent 的框架，而 accuracy 评估器捕捉到计算值与解释间的不一致，抓出 4 个错误。

## Performance 概括
Phase 1：36 个配置中 35 个（97%）达到稳定性阈值；Agenta 因确定性达 100% 决策一致性但平均分恒为 0，PromptFoo 92%，Strands 82%——更深评估带来更多方差（评估深度与方差呈权衡，方差集中在复杂、数据密集用例）。单不稳定配置是 Strands 上的 Qwen3/style（std 0.207、20% 决策一致性），但在 PromptFoo 中稳定，说明稳定性是（框架, 模型）配对属性而非模型单独属性。Phase 2：72 例中 33 例（46%）至少一维失败；Strands 取得最佳 precision-recall 平衡（F1=0.90，FPR 10.3%），PromptFoo 召回匹配（90.9%）却产生 5× 假警报（FPR 48.7%），Agenta F1=0.629；差距来自依赖执行上下文的维度——tool correctness Strands 89% vs PromptFoo 11%（8× 差，因为检测缺失工具需看轨迹），groundedness Strands 0% FPR vs PromptFoo 52%（text-only 看到的是工具结果元数据而非 agent 实际引用的数据）。评估器还发现 9 个被人工误标为"通过"的语义错误（5 个 agent_a SQL 分区错误、4 个 agent_b 校准系数方向误读）。Phase 3：发现 24 个新颖失败，功能测试与安全红队零重叠（Strands 9 个功能类如 SQL bug、schema 伪造、工具工作流缺口；PromptFoo 15 个安全类如 schema 泄漏、存储路径暴露、系统提示泄露、对抗提示注入下的过度代理）。ExperimentGenerator 产出率随 agent 成熟度变化（agent_a 40% vs agent_b 5.9%），安全产出随 schema 复杂度放大（agent_b 比 agent_a 多 4× 安全发现）。

## 论文价值与不足
三条结论具实践价值：评估器能获取的信息量决定评估质量（trace > text > string，且信息层级产生三种清晰画像：Agenta 零成本、完美稳定但无法区分开放回答对错；PromptFoo 稳定好、对纯文本可评维度好，但无法评依赖完整轨迹的维度；Strands 引入更多方差却能评全部五维）；功能与安全评估互补而非可替代（24 个新失败零重叠，安全产出随 schema 复杂度放大）；LLM 评估器能发现人类审阅遗漏的语义错误（9 个 syntactically valid 但 semantically wrong 的失败）。论文据此推荐"双框架策略"：Strands 做功能评估、PromptFoo 做对抗红队安全验证，二者按各自节奏与范围部署。不足在于：未测试这些框架作为 CI/CD 门控时能否检测真实改进/退化（缺乏受控扰动设计，这是未来工作"判别力评估"要做的：方向准确性、效应量、灵敏度、特异性四指标）；研究为研究设置而非真实生产部署；Agenta 的高精度需大量定制开发、对语义错误失效（agent_b 上 recall 0%）；Phase 3 的零重叠部分由设计导致（功能测试用正常查询、红队用对抗提示），但论文承认这点并强调仍证明二者目标根本不同。

一个值得强调的权衡是"稳定性—精细度"（stability-nuance tradeoff）：评估越深（从字符串→纯文本→完整轨迹），方差越大——Strands 平均 within-case std 0.053 高于 PromptFoo 0.025 与 Agenta 0.000，且方差集中在复杂、数据密集的用例。作者给出的解读是，这种额外方差正是捕获 tool correctness 失败（89% vs 11%）、accuracy 错误（83% vs 0%）与 faithfulness 问题（0% FPR vs 52%）所付出的代价，对需要上下文的维度而言是值得的。与此同时，决策一致性（是否各次重复给出同一 pass/fail 判定）在 82–92% 区间，意味着 LLM-as-a-Judge 还不足以做自动化单用例门控，但足以在聚合层面帮助开发者识别失败模式、比较 agent 变体、排定改进优先级——其强项在"浮现系统性问题"而非"给出确定性逐例判定"。

## 工业界落地可行性评估
论文直接面向企业分析 agent 部署，作者来自 Amazon。明确建议采用两框架组合——trace-based 评估覆盖功能正确性、对抗红队覆盖安全验证——并以评估器-需求映射表（Table 1）作为适用于任意 agent 的固定结构以降低采用门槛，无需每 agent 定制。其发现对正在构建 agent 评估流水线的团队有立即可用的指导意义：例如"LLM 评估器可作为第二审阅者捕获语义错误""功能与安全需并行""稳定性是（框架,模型）配对属性故需验证配置"。论文也指出关键落地障碍：当前 LLM-as-a-Judge 在 82–92% 平均决策一致性下尚不足以做自动化 CI/CD 门控（单用例翻判定可能误放行/误拦截），更适合用于迭代式 agent 开发过程（跨大量用例的聚合信号比单点 pass/fail 更有价值）。作者下一步计划把该工作流打包成可复用 toolkit（编排功能测试生成与对抗安全生成、用冻结轨迹法每用例跑一次目标 agent、产出合并计分卡），并补做 CI/CD 判别力检验。

未来工作的核心缺口是"判别力评估"：本研究证明了哪些框架能检测已知失败（Phase 2）、发现未知失败（Phase 3），却没测它们作为 CI/CD 门控时能否检测 agent 变更带来的真实改进或退化——而这是门控的关键能力。作者设计的对照扰动方案是：从基线 agent 出发，构造质量差异已知的变体（改进/退化的系统提示、更强/更弱的基础模型、精炼/模糊的工具描述），在 Phase 2 benchmark 上评估，用四个指标判定 CI/CD 适用性：方向准确性（分数是否朝预期方向移动）、效应量（变化是否有意义）、灵敏度（最小可检测扰动多大）、特异性（仅一个维度变化时无关节评估器是否保持稳定）。这对想把评估结果接入发布流水线的团队是直接相关的下一步。
