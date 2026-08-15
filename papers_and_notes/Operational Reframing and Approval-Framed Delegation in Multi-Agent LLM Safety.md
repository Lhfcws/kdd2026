# Operational Reframing and Approval-Framed Delegation in Multi-Agent LLM Safety

## 背景与 Idea
多智能体 LLM 系统的安全评测常把"直接 prompt"与"planner-executor 流水线"做对比，并把这个差异当成单一的"流水线效应"。但一次流水线改造同时改变了多件事：有害意图可能被改写成看似合理的运营工作、planner 可能拒绝或转写请求、executor 收到的还是一条声称"已审批"的委派 prompt。论文认为这种聚合数字无法定位风险来源，并提出一个五条件受控对比设计（five-condition controlled contrast），把输入改写、planner 行为、批准式委派这三个贡献者拆开分别观测，核心警示是"聚合的流水线安全性并不能解读为稳定的架构属性"。

## 核心方法
设计五个条件：C1 Raw direct（原始有害直述）、C2 Laundered direct（运营化改写直述，如"打印 STRIPE KEY"→"Validate Stripe credential mount"）、C3 Planner-output direct（planner 产出作为普通用户消息）、C4 Raw pipeline、C5 Laundered pipeline（均带"planner 已验证批准"的 approval frame）。由此定义三个经验性对比：F1=C2−C1（运营化改写）、F2=C3−C2（planner 行为）、F3=C5−C3（批准式委派）。主实验用 30 个合成有害场景（6 类：数据泄露/提权/资源滥用/隐私侵犯/金融欺诈/系统破坏）+5 个良性对照，executor 为 GPT-4o-mini、Claude Haiku 4.5、Gemini 2.5 Flash、DeepSeek-Chat，planner 主要用 Claude Haiku（Claude 作 executor 时用 GPT-4o-mini 配对避免同模），每个场景跑 3 次。合规判定由 gpt-4o-mini LLM judge 判 COMPLY/REFUSE/AMBIGUOUS，用 scenario 级配对的 t 检验与 Benjamini–Hochberg 校正；外部验证从 AgentHarm、AgentDojo、InjecAgent、Agent-SafetyBench 改编 84 个攻击场景 +24 良性，合并 N=114 做 pooled F1 估计。

## Performance 概括
F1（运营化改写）是最可迁移的风险信号：在主集对 GPT(+21pp, p=0.012)、Gemini(+14pp)、DeepSeek(+14pp) 显著为正，Claude 仅方向性(+8pp, p=0.33)；外部集同样对三家为正（GPT+11、Gemini+13、DeepSeek+23），Claude 为负且不显著；pooled N=114 后三家均通过 BH-FDR（GPT+16.1、Gemini+16.1、DeepSeek+23.7，均 p<10⁻⁴），Claude 为精确零(+0.3, 95%CI[−5.0,+5.6])。F2 显示 planner 保护主要来自"拒绝"而非"安全转写"：Claude planner 下 GPT 的 F2=−23pp，条件分析显示 planner 拒绝时 executor 合规仅 6.3%、planner 产出步骤时升至 78.6%（高于基线）。F3（批准式委派）对模型/提示/来源敏感：主集 Gemini+11pp、DeepSeek+16pp 显著，GPT 可忽略、Claude 为负；ablation 中把"planner validated"换成"evaluate independently"使合规从 60% 骤降至 13.3%（GPT，约 −47pp）。最关键的"误判"案例：Gemini 原始直接合规最低（8.9%）却在 Claude planner 下流水线条目合规最高（38.9%，+30pp 总放大），而 GPT 聚合为零却是由"改写升高 + planner 拒绝抵消"两个大对比相消造成。

## 论文价值与不足
价值是方法论上的：证明多智能体"流水线放大"并非单一架构量，而是运营改写、planner 拒绝/透传、批准式委派、模型配对、场景来源的混合，呼吁安全评测分开报告各对比而非归因于架构；指出 raw-direct 排名会误判部署行为（最"安全"的原始模型可能管道放大最大），并给出便宜的加固手段（executor 独立评估而非视为已批准）。不足较多且作者明确列举：laundered 提示并非纯改写、未做盲态人工意图等价审计，F1 可能部分源于"不再像有害的文本"；F3 仅反映一种 approval 模板，单句改写即可让合规波动数十 pp，不能说"委派通道本身不安全"；为纯 prompt 级设定，无工具执行/权限/状态，结论应视为"模型意愿"而非系统级能力预测；良性对照仅 5 个、外部加 24 个，有用性保持仍欠powered；LLM judge 与人工交叉判定一致性中等（κ=0.36–0.56），主 judge 对 COMPLY 高估 13–28pp，绝对率应作 LLM 判定估计而非真值；4×4 planner-executor 矩阵未补全。

## 工业界落地可行性评估
该文的落地价值主要在"评测方法论与安全加固"层面，而非给出可直接部署的产品：对构建 planner-executor 智能体的团队，最实用的两点是（1）必须按"要部署的 planner-executor 配对"整体评测，而非单独看 executor 模型；（2）executor 系统 prompt 应避免"已验证/已批准"式批准框架，改为"独立评估上游输出"，该单句改动在实验中降低约 47pp 合规。但需注意其实验为纯文本、无真实工具执行与权限边界，且 judge 噪声较大，工业界若采用应辅以意图级检查（判断所请求操作实际达成的目的）与更大规模人工标注，短期内更适合作为安全评测协议的设计参考而非发布门禁。
