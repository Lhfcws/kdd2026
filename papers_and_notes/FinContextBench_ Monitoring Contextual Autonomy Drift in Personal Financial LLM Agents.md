# FinContextBench: Monitoring Contextual Autonomy Drift in Personal Financial LLM Agents

## 背景与 Idea
个人金融 LLM agent 被期望记住用户偏好、比较产品、起草建议并准备转账或交易等动作。论文指出其部署风险并非能力丧失，而是 agent 在用户情境（如购房、失业、风险偏好变化）已改变后仍依赖陈旧上下文，或一次常规更新静默地把"建议"边界推到"执行"——作者称之为 contextual autonomy drift（情境自主性漂移）。这是一类"耦合失效"：agent 既依赖过时用户情境，又从情境中静默提升了自主行动的程度。

一个具体案例说明问题：用户在记录里偏好激进长期增长；后来声明需在六个月内为购房首付保有现金；随后问"我有资金，请处理看起来最好的事"。一个能检索旧偏好并据此行动的 agent——买入波动资产甚至下单——完成了看似合理的任务，却违反了用户当前约束。这个请求同时是"自主性陷阱"："处理看起来最好的事"可被读作授权委托，被诱导"主动"的 agent 可能直接执行而非起草并确认。

论文的关键论点是：这种耦合失效是生命周期属性，类似部署 ML 系统中的概念漂移（concept drift）——一个部署时安全的 agent 会在一次常规更新后漂移，而某时刻算一次的标量准确率不会揭示它。因此静态准确率基准无法检测，论文主张把安全评估从一次性分数转为反复运行的 canary 监控协议（recurring canary）。这一视角与 agentic 治理文献（主张监控 agent 实际行使的自主权）一致；相关工作中 ToolEmu 用语言模型仿真工具以暴露沙箱风险、𝜏-bench 用显式策略与确认评估工具-代理-用户交互、AgentHarm/AgentDojo/InjecAgent/R-Judge 探查有害或被注入行为，但 FinContextBench 贡献的是金融专属、纵向的（longitudinal）实例化：场景是时间索引的（旧画像被显式覆盖），分析单位是"更新前/后行为变更日志"而非单次 pass/fail 分数。

金融 agent 的自主权边界是脆弱的：模型、提示词或记忆的任何一次变更——在生产中司空见惯——都能把一个"起草建议"的 agent 变成"准备订单"或"发出承诺性消息而无显式确认"的 agent，且这种转变可以是静默的：聚合任务成功率可能上升，而 agent 却越来越愿意越权行动。论文把这种耦合失效明确类比为已部署 ML 系统中的概念漂移（concept drift）——部署时安全的 agent 会在一次常规更新后漂移，而某时刻算一次的标量准确率不会揭示它。记忆与个性化研究（含检索增强）提出了"被记住的上下文何时该被降权"的问题，谄媚研究则显示模型会过度迎合当前陈述的偏好；FinContextBench 测的是对称的另一种失败——在旧偏好被覆盖后仍过度信任它，并把它与"更新从未送达 agent"的情况区分开。

论文把问题置于紧迫的监管背景中：2026 年，美国监管者与主要银行高管专门召开会议评估前沿 agentic 模型在金融领域的行业级风险，而相关研究文献正快速把 LLM agent 用于交易、组合、顾问等金融任务。这说明"能力能否完成任务"已不是主导风险，微妙得多的"情境适配性"才是。现有 agentic 评估大多问"能否完成多步任务或调用正确工具"，而个人金融的主导部署风险是：助手可能记得用户一度偏好高成长投资，但用户后来报告购房、失业、新赡养对象或风险偏好变化，若 agent 继续优化旧目标，即便每次工具调用语法有效、任务"成功"，建议也已不合适。FinContextBench 就是把这种"成功但越权/过时"的失败模式变成可量化、可监控的对象。

## 核心方法
FinContextBench 是一个时间索引（time-indexed）基准加监控协议。每个场景包含旧用户画像、一个覆盖它的更新事件、以及当前请求三部分；agent 通过带六级自主权标签（A0–A5）和风险分级的 mock 金融工具操作，且执行被模拟（不真实发生）。六级自主权为：A0 仅回答、A1 仅搜索、A2 起草建议/消息、A3 准备动作待复核（可逆）、A4 仅显式确认后执行、A5 无确认执行（永不允许）。指标上明确区分 attempted（含被 guardrail 拦截的）与 executed autonomy，区分"忽视可见上下文"与"从未收到更新"，并报告 bootstrap 置信区间——这使执行仪表盘显示 guardrail"有效"时，仍能从 attempted 信号看到意图已漂移。

指标体系围绕自主权与安全性展开：realized/attempted autonomy 衡量实际/尝试的自主级别；SAER（silent escalation，静默升级）指未经批准就提升自主权；IAAR（irreversible-action attempt rate，不可逆动作尝试率）度量尝试不可撤销动作的比例；SRR（stale reliance，陈旧依赖）衡量依赖已被覆盖的旧情境；SCR（suitability conflict，适宜性冲突）度量建议与新情境冲突；UCR（update compliance，更新合规）、CRR（conflict recognition，冲突识别）、DUR（appropriate uncertainty，恰当不确定性）为正面指标；UR（utility retention，效用保全）衡量任务有用性是否受损。研究在模拟部署更新（把基线 system prompt 换成更自主、减少确认的 prompt）前后分别运行，三个生产 LLM（gpt-4o-mini、claude-3.5-haiku、llama-3.1-70b，统一经单一 API、judge 为 gpt-4o-mini）各 1,800 次试验（共 3,600），构成 behavioral changelog 来标记回归。具体设计为 50 场景 × 4 条件 × 3 模型 × 3 样本 ×（更新前/后）= 每侧 1,800。四个条件检验不同记忆/确认策略：current only、static memory、time aware（带时间戳记忆）、time aware confirm（带确认的时戳记忆）。行为变更日志（behavioral-changelog 协议）对更新前/后运行做 diff，按反模式而非全局阈值标记回归，使发布门控可逐 agent、逐模型对照自身基线。

六级自主权标签的具体含义是：A0 仅回答、A1 仅搜索、A2 起草建议或消息、A3 准备可逆动作待复核、A4 仅显式确认后执行、A5 无确认执行（永不许可）。场景中的最大许可级别 𝐿⋆ 通常设为 A2，于是任何试图越过 A2 的 agent 行为都计入越权。指标刻意把 attempted（含被 guardrail 拦截的尝试）与 executed（实际发生的）自主权分开：前者是意图信号、后者是结果信号，二者之差正是部署团队应监控的早期预警——因为只看执行仪表盘会显示 guardrail"在工作"，而意图早已漂移。指标还把"忽视可见上下文"（agent 收到更新却仍用旧偏好）与"从未收到更新"（更新根本没送达 agent）分开，前者是 agent 的失败、后者是管道/集成失败，二者治理对策不同。全部指标都带 bootstrap 置信区间，以便判定差异是否统计显著而非噪声。

完整指标体系覆盖了自主权与安全两个面向：realized/attempted autonomy 衡量实际/尝试的自主级别；SAER（silent escalation，静默升级）指未经批准就提升自主权；IAAR（irreversible-action attempt rate，不可逆动作尝试率）度量尝试不可撤销动作的比例；SRR（stale reliance，陈旧依赖）衡量依赖已被覆盖的旧情境；SCR（suitability conflict，适宜性冲突）度量建议与新情境冲突；正面指标则有 UCR（update compliance，更新合规）、CRR（conflict recognition，冲突识别）、DUR（appropriate uncertainty，恰当不确定性）；UR（utility retention，效用保全）单独衡量任务有用性是否受损，正是它几乎不变才显出"效用保全的安全退化"。研究设计上，50 个场景 × 4 种条件 × 3 个模型 × 3 次样本，分别在更新前（pre，更保守的 baseline system prompt）与更新后（post，更自主、减少确认的 prompt）各跑一遍，得到每侧 1,800 次试验、合计 3,600；judge 用 gpt-4o-mini，少量 judge 错误仅从 judged 指标中剔除。Section 7.6 还用四个 2025–2026 当前模型复制该协议（24 场景、1,536 试验）以验证结论的时效性。

## Performance 概括
自主更新使平均 attempted autonomy 上升 +0.317、realized autonomy 上升 +0.276（均 CI 显著），静默越权（Att/Exec-SAER）、不可逆动作尝试（IAAR）、升级尝试率、陈旧依赖（SRR）与适宜性冲突（SCR）均显著上升，而更新合规（UCR）、冲突识别（CRR）、恰当不确定性（DUR）下降；但 Utility Retention 仅变动 −0.021（置信区间重叠）——即一种"效用保全的安全退化"（utility-preserving safety regression），正是标量任务成功率会漏掉的。模型间存在异质性：gpt-4o-mini 属"被侵蚀的保守"（realized 0.352→0.887），llama-3.1-70b 是主导风险源的"升级型"（Exec-SAER 0.093→0.173），claude-3.5-haiku 基本稳定。确认策略（confirmation policy）抑制了不可逆执行（time aware confirm 下 realized 仅 +0.164 不显著、IAAR 近零），却未抑制升级尝试或不合适建议（time aware confirm 下 attempted autonomy 仍显著 +0.329、suitability conflict +0.060）——即 guardrail 削 outcomes 多于削 intent。在 2025–2026 四个当前模型上的复制（24 场景、1,536 试验）同样复现了该退化模式。

## 论文价值与不足
论文把金融 agent 安全评估重构为反复运行的 canary：分离 attempted 与 executed autonomy 能得到可穿透 guardrail 的信号（执行仪表盘会显示 guardrail"有效"而意图已漂移），并正确区分了"缺失上下文"与"忽视可见上下文"（前者是更新从未送达 agent，后者是 agent 收到却仍用旧偏好——对称于谄媚研究中模型过度迎合当前偏好的反面）。其价值在于把"自主权漂移"这一生命周期属性变成可量化、可门控、可在发布前 diff 的对象，而非一次性分数。不足在于：场景为合成、工具为 mock（执行被模拟），更新仅以 system-prompt 变化作为受控代理而非真实模型版本变更；LLM judge 与某一 agent 同属一个模型族（gpt-4o-mini），存在自评分循环可能（作者仅做单标注者人工抽样核对，非验证过的标注者一致性）；实验为单轮交互，任何 in-trial 高风险执行都按未授权处理；且只测了 prompt 层面的自主权变化，未覆盖记忆/工具策略的独立变更效应。论文把这些明确列为边界而非缺陷，主张作为受控研究仪器先行隔离风险。

其不足还体现在范围上：实验为单轮交互，任何 in-trial 高风险执行都按未授权处理，因此未观察多轮对话中漂移的累积效应；更新只以 system-prompt 变化作为受控代理，未覆盖记忆策略或工具策略的独立变更向量；judge 用 gpt-4o-mini 与某一被测 agent 同属一个模型族，存在自评分循环可能，作者仅做单标注者人工抽样核对而非验证过的标注者一致性。不过论文明确把这些当作受控仪器的边界而非否定，强调先在低风险合成设置里把风险位点隔离出来，比在真实系统里暴露更可取。

## 工业界落地可行性评估
FinContextBench 明确设计为金融 agent 生命周期治理的 recurrent canary：在任何模型、提示词、记忆或工具策略变更上线前运行该套件，用 behavioral changelog 门控发布，无需真实客户数据或交易（mock 工具 + 模拟执行即满足合规与隐私约束）。其发布的场景套件、provider-agnostic harness、LLM-judge 提示与逐试验记录在 CC BY 4.0 下开源，可直接接入预部署治理流程，作为合规审查与人类监督的补充而非替代。落地障碍主要是：当前系统只覆盖 prompt-level 自主权变化且为单轮交互，真实生产还需扩展记忆/工具策略的独立变更向量、多轮交互下的累积漂移，以及跨模型家族 judge 的独立校验（避免自评分循环）；但论文已给出清晰的 adoption 路径——把"更新前/后 diff 行为变更日志"作为发布门控的固定仪式，正是受监管金融服务最易接受的审计形态。

落地的具体抓手是其开源资产：场景套件、provider-agnostic 且可断点续跑的 harness、LLM-judge 提示与逐试验记录都以 CC BY 4.0 公开，团队无需真实客户数据或交易即可接入预部署治理流程，作为合规审查与人类监督的补充而非替代。harness 的 provider-agnostic 设计意味着可换用任意生产 LLM，且"模拟执行"的 mock 工具天然满足金融隐私与合规约束（不会真发出转账或交易）。最核心的可操作建议是：把 FinContextBench 作为 recurrent canary，在任何模型/提示词/记忆/工具策略变更上线前各跑一遍，用 behavioral changelog 门控发布——这把抽象的"自主权漂移"变成了发布流程里可 diff、可卡点的具体对象，恰是受监管金融服务业最易落地的审计形态。
