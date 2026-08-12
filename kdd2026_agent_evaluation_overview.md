# Agent 测评综述：当前大家在讨论什么

> 素材来源：本工作区 47 篇 KDD 2026 论文速读笔记中，标签含「测评」的 41 篇（index.md 中序号 #1–#15、#18–#25、#27–#40、#42、#45–#47）。以下结论均提炼自各篇笔记正文（背景与 Idea / 核心方法 / 价值与不足），不引入外部信息。
> 说明：这批论文高度集中在"如何评测 agent"——"测评"是绝对主线（41/47）。因此这份综述本质上是在回答：**当社区说"我们要更好地评测 agent"时，具体在解决哪些子问题。**

---

## 一、一句话结论

Agent 测评正在从"跑个 benchmark 看准确率"转向"评过程、要可复现、把安全与成本拉进评测一等公民"。大家不再满足于一个标量结果，而是追问：这个结果可信吗？过程合规吗？协议可复现吗？成本可接受吗？安全/护栏被忠实执行了吗？——一句话，**评测对象从"答案"扩展到"整个 agent 运行的全生命周期与可信属性"**。

---

## 二、五条跨主题趋势

1. **从只看最终结果 → 评过程 / 轨迹 / 生命周期**
   评测对象从标量答案扩展到推理轨迹效率、事务状态完整性、行为纪律与全生命周期 trace-backed 归因（#7 TRACE、#28 ACID-Bench、#40 企业分析 agent、#46 经济系统、#13 ML 实验生命周期）。

2. **从 LLM 主观打分 → 确定性 / 可执行 oracle + 统计严谨**
   用执行引擎、状态机、SQL、单位矩阵保真度等"零方差 oracle"替代 LLM 主观判分，并用噪声地板、最小任务预算、协议规范保证结论可复现（#6 量子电路、#22 PCBWorld、#28、#32 知识图谱、#15 Protocol Cards、#45、#29、#8）。

3. **从单一 benchmark → 构念有效性审计 + 噪声地板门禁**
   排名不再被盲信，而是审计其收敛性/区分效度、证明排名稳定性，并要求"协调增益""自改进"等声明必须跨越配对的测量噪声地板（#23 构念效度失败、#38、#25 MoltBook、#29、#10）。

4. **评测与成本 / 效用 / 预算绑定**
   可信赖性被定义为"质量 × 可靠性 × 成本"的联合属性；预算感知的效用、校准、延迟成为评测一等维度（#5 稳定币、#6、#42 Active RAG、#36 token/solved、#35 路由成本）。

5. **安全 / 可信与多智能体特殊性成为评测一等公民**
   动作准入、护栏忠实、能力–安全联合审计、red-teaming，以及均衡级危害 / 共识过度自信 / 协调噪声等系统级失败，被系统纳入评测框架（#1、#3、#4、#9、#11、#12、#14、#20、#21、#24、#27、#31、#35）。

---

## 三、主题分述（9 个方向）

### 主题 1：过程 / 轨迹 / 生命周期级评估（超越最终答案）
**痛点**：最终答案相同，但推理轨迹、状态完整性、行为纪律可能已失败；只看结果无法定位根因，也发现不了"成功但有害/违规"的运行。

- **#7 Beyond the Final Answer（TRACE）**：用 reference-free 的 evidence bank 评估推理轨迹的"效率 / 幻觉 / 适应性"三维度，无需 ground-truth 轨迹即可监控线上 tool-augmented agent 的冗余与幻觉步骤。
- **#13 Governed ML Experimentation**：用 append-only ledger 记录实验全生命周期，以 provenance completeness（来源完整性）与 failure taxonomy 把可审计性前置到过程本身。
- **#28 ACID-Bench**：对状态变更工具 agent 做确定性故障注入（部分提交 / 幽灵成功 / 过期读 / 崩溃恢复），用基于状态的评估器分离"任务成功"与"事务完整性 / 安全交接"，刻意不用 LLM judge。
- **#40 Enterprise Analytics Agents**：端到端 trace-backed 方法论，三信号族（语义理解 / 执行质量 / 可靠性）把失败归因到具体阶段而非仅看最终答案。
- **#46 Outcome Is Not Enough**：经济 agent 的 trace-based 生命周期评估，提出 discipline stability（纪律稳定性）——标量 RevPAR 相同但行为纪律可能失败，用 D1 距离 / JS 散度监控价格 / bid 分布偏离。

### 主题 2：构念有效性危机与审计
**痛点**：benchmark 是否真在测量它声称的"构念"很少被检验，排名背后的可靠性存疑，单平台 / 单模型结论易过度外推。

- **#23 Construct Validity Failures**：用心理测量学跨 benchmark Spearman 相关与排名反转审计 5 个主流 benchmark，平均 ρ=0.67、22% 模型对存在排名反转，提出 D1–D5 desiderata，结论"无一个 benchmark 满足全部五条"。
- **#25 Emergent, Steered, or Neither?**：对 MoltBook agent society 做有效性审计，用度保持零模型 + 经外部真值校验的自主 / 人类账号判别，证伪"幂律级联 / 社会强化 / 协作增益"等被广为引用的论断。
- **#38 Auditing Construct Validity（Sports TRACE）**：把测量科学的构念效度四威胁（T1–T4）操作化为可度量审计工具（APCR / IAJA / SRI / MERS），实例化 PressureBench 审计足球战术分析论断，得出"单次 producer 调用不安全""小模型 FP 更低"等反直觉结论。

### 主题 3：评估的统计严谨性、可复现性与协议规范
**痛点**：评测协议、harness / scaffold 等隐藏变量与测量噪声使结论不可比、不可复现，单 seed 显著结果常在第二 seed 瓦解。

- **#8 MacCode Lab**：单机构可复现 harness，揭示 public-test 与 hidden-test 重打分可能系统性分歧（7B 模型衰减远重于 35B），且 repair 是否有用取决于模型而非任务。
- **#15 Beyond Leaderboards（Protocol Cards）**：指出评测协议本身是隐藏变量，以 SNRfile + clean-apply rate 让 coding-agent 评测可复现可审计（full-raw 比 diff-only 干净应用率 8% → 54.7%）。
- **#29 How Much Coordination Gain Is Real?**：提出"协调噪声地板"配对协议作为协调架构声明的发布门禁，发现 7/10 近期多 agent 架构报道的提升低于本地噪声地板，单 seed 显著结果在第二 seed 即瓦解。
- **#36 The Scaffold Effect**：harness 选择是隐藏混淆变量，token / solved 差异可达 40×，提出以 **harness–model pair** 为评估单位并报告失败指纹（REASON / VERIFY / TIME …）。
- **#45 How Many Tasks Are Enough?**：replay 分析提出"最小充分任务预算"，证明部分评估必须声明阈值 / 覆盖规则 / 未解决比较数，否则会隐藏错误决策（SWE-bench Lite 在 95% 预算前均未达标）。

### 主题 4：rubric / LLM-judge 方法论（verifier 设计、judge 选择、rubric 优化）
**痛点**：LLM judge 选择本身就在改写评估结论；rubric 措辞决定人-AI 共识；高成本 judge 并非必要。

- **#10 Comparative Analysis of Agent Eval Frameworks**：对比 trace-based / text / 确定性启发式三范式，发现信息量（trace > text > string）决定质量，功能与安全评估零重叠（推荐双框架策略）。
- **#18 Verifying Agents in Rubric-Graded Environments**：从 rubric 反推九大 verifier 能力与 P1–P3 原则，Gandalf 以 Pareto 占优、最便宜配置（F1 0.633 / $42）超过最贵 baseline（F1 0.538 / $414）。
- **#30 Selecting LLM Judges**：量化 judge 选择的 accuracy masking——side-effect 类错误 judge 间差异高达 45–50pp，task-success 仅 1.6–4.4pp，提出高召回 judge + 复核负担权衡。
- **#33 Clustering-based Prompt Optimization**：稀缺标签预算下用 cluster + batch-validate 一次性优化 judge rubric，约 $25 即在留出集稳健提升人-AI 共识（P(Δ>0)=100%）。
- **#34 Trustworthy Rule Learning**：用错误驱动规则归纳 + 规则演绎做可审计决策，规则库作为"一致性锚点"消除 LLM 随机性（92% → 84%），CoT 兼作评估工具。

### 主题 5：确定性 / 可执行 oracle 与 benchmark 自动化构建
**痛点**：依赖 LLM 主观评判不可复现、有方差；benchmark 构建昂贵且词汇爆炸；需要"零方差、可纵向监控"的 oracle。

- **#6 Quantum Circuit Vision**：以单位矩阵保真度（unitary fidelity, F≥0.99）作不依赖 LLM 的客观 oracle，成本感知评估"视觉→量子代码"agent，并给出 Haiku→Sonnet→Opus 级联路由的成本分析。
- **#19 Evaluating Agentic Skills at Scale**：自动生成约 1000 任务覆盖 500 真实 skills 与 19 个 agent-model 配置，提供规模化、可复现的技能评测排行榜。
- **#20 NRT-Bench**：控制室多轮 red-teaming，用 **judge-free** 的 ASRCSF（违反关键安全功能成功率）作主指标，避免主观评审偏差（8.7%–12.1% 会话致至少一个 CSF 丢失）。
- **#22 PCBWorld**：engine-grounded EDA 基准，以真实 KiCad 引擎作 oracle 校验"能否制造"，八项引擎可校验指标（Clean Pass 为主指标）。
- **#32 Diagnostic Knowledge Graphs**：用同一规范化知识图谱同时解决 benchmark 构建与 agent 推理（评估–agent 对偶性），SQL-grounded 确定性精确匹配实现**零轮间方差（σ=0）**。

### 主题 6：安全与可信纳入评测
**痛点**：可信 agent 输出可能流畅、合理却不可执行 / 不安全；安全需从"事后安全测试"升级为"评测的一等公民"（动作准入、护栏忠实、能力–安全联合）。

- **#1 MADS-CPS**：run-level 机器可校验评估契约，定义"单条运行是否可被审计 / 回放 / 发布"的准入证据对象与 fail-closed 的 PONR 发布门禁。
- **#3 ReflexBench**：提出 observer-participant failure（输出改变被评估环境 / 未来观测 / benchmark 本身），用 observer depth 四级量规 + 退化差 ΔOD 度量 agent 对自身因果足迹的感知。
- **#4 Evaluating Admissibility**：把"动作是否可准入"定义为需 Justification + Authority + Replayable 的治理检查，在会计记账域做高后果压测。
- **#9 FinContextBench**：contextual autonomy drift canary——分离 attempted 与 executed autonomy，发现"效用保全的安全退化"（标量任务成功率会漏检的静默越权）。
- **#11 Operational Reframing**：五条件受控对比拆开运营改写 / planner 行为 / 批准式委派，证明聚合的流水线安全性不能解读为稳定架构属性（Gemini 原始合规最低却管道放大最大）。
- **#12 Numerical Format as Risk Surface**：首个 FP4 VLM 能力–安全联合审计，证明量化格式切换是权重哈希 / 提示监控**不可见**的部署后风险面（拒绝率可无声移动 10–18pp）。
- **#14 SolarChain-Eval**：把物理可行性（Physics Gate）建模为可信经济 agent 的硬门槛，抑制"注水式流动性刷分"。
- **#27 Confidence-Aware Orchestration**：置信度感知多智能体编排做多模态规则合规评估，跨模态冲突解决机制提升 F1 约 13.9pp。
- **#31 Guardrails as Scapegoats**：审计不忠实安全拒绝，56.6% 拒绝实为把 guardrail 当替罪羊的模型自身偏差（safety-framed 措辞使虚假拒绝 15.6× 上升）。

### 主题 7：多智能体评估的特殊问题
**痛点**：个体评估会系统性漏检集体 / 均衡 / 共识层面的失败；多 agent 的协调增益、一致性、均衡危害需要专门评估协议。

- **#21 Equilibrium-Level Harm**：定理证明仅基于固定非战略输入的个体评估必然存在"个体通过却产生均衡级危害"的游戏，给出三个结构性风险预测因子。
- **#24 When Consensus Is Not Correctness**：揭示 debate 一致性是内生"过度自信制造"（ρ̄ 0.53 → 0.96，单看一致性 AUROC 仅 0.49–0.58），提出 split-conformal 的 Certify（Affirm）替代"一致即停"。
- **#35 Adaptive Multi-LLM Orchestration**：replay-based 评估协议固定每个"任务–agent"对的潜在结果，把路由策略比较从随机 LLM / judge 噪声中剥离。
- （交叉：**#11 / #27 / #29** 亦属多智能体安全 / 协调评估。）

### 主题 8：成本 / 效用感知评估
**痛点**：仅看准确率忽略推理成本、预算、校准；生产部署必须权衡效用–成本–延迟。

- **#5 StableEval Arena**：成本感知稳定币风险监测 benchmark，联合报告预测质量 / 运行可靠性 / 推理成本，发现 agent 执行可靠（valid-JSON 1.0）但稀有压力检测极弱（stress recall 最高仅 0.143）。
- **#42 When Should Active RAG Retrieve?**：把 Active RAG 触发重构为预算效用估计，量化 benefit（31.9%）/ harm（4.0%）与固定 50% 阈值的校准违规（28.6%–65.7% 不同 split）。
- （交叉：**#6** 单位保真度 + 级联路由边际成本；**#35** 路由成本 / 截止命中；**#36** token / solved 成本信号。）

### 主题 9：智能增强 / 自改进系统的评测
**痛点**：自改进系统缺少可靠 ground truth、修复质量难以审计、聚合对齐可能误导。

- **#2 On Recursive Resolution**：用"LLM 与人工一致即免费信号、分歧才升级"打破 ground truth 自举悖论，以分层 Wilson 区间 + 五层递归闭环把评估结论反哺修复（标注工时降 15.95×）。
- **#39 Autoresearch for Marketplace Catalogs**：生产级自研究循环（propose-evaluate-keep）评测目录重建，跨家族 judge / critic 避免偏见互相强化，强制人工 sign-off 才上线。
- **#47 When Aggregate Alignment Misleads**：揭示"聚合对齐误导"——policy 边际动作分布像基准却轨迹级组成与收入失败，提出多粒度行为审计 + shuffled-diagnostic placebo 对照协议。

---

## 四、空白与开放问题

1. **智能增强 / self-improvement 的评测仍偏合成、缺真实部署验证**：#2 / #39 / #47 多基于合成模拟器或单垂类、单模型、单生产运行，缺乏真实部署时间线与长期分布漂移验证；"完全自主修复""per-state 专家动作""跨域因果对比"均为 future work。

2. **记忆与检索评测明显薄弱**：#37 显示通用记忆系统在 BIM 专业工作流准确率 <60% 且仅单模型、合成数据；#42 的 Active RAG 仅用 Qwen2.5-1.5B 单模型、单任务。长程记忆跨会话复用、RAG 触发决策的校准仍是弱项（记忆类论文全部与"测评"共现，说明它更多是"被评估对象"而非独立方法主线）。

3. **跨 benchmark 可比性与构念效度仍未决**：#23 显示 22% 模型对排名反转、无 benchmark 满足全部 D1–D5；#25 / #38 审计仅限单平台 / 单模型家族；持续审计机制与跨域泛化尚未建立。

4. **动态 / 在线评测与真实部署校准不足**：绝大多数工作在合成环境、冻结模型、单 seed 下验证（#46 / #47 用合成 simulator；#1 / #12 缺真实硬件 / 真实工具执行；#11 / #31 为纯 prompt / 合成 stub）。#9 的 canary、#46 的监控栈指明方向，但真实流量下的持续评测与校准仍待验证。

5. **judge 选择与 rubric 的质量 / 可移植性开放**：#30 揭示 judge 选择可造成 side-effect 类 45–50pp 的 accuracy masking；#33 / #18 仅在单垂类 / 金融域验证，缺跨域通用 judge 选择准则；LLM judge 的噪声、校准与容量伪影（#12 的 judge 容量消融）仍需系统研究。

6. **多智能体均衡级 / 涌现危害可测但难缓解**：#21 定理证明个体评估必然漏检均衡级危害，#24 证明一致性是过度自信制造，#29 显示多数协调增益实为噪声——三者均缺乏开放环境下的低成本干预方案，且多在合成博弈 / 单模型家族验证。

7. **协议 / harness / scaffold 作为隐藏变量尚未被工业界普遍采纳**：#15 / #36 揭示协议与 harness 是强混淆变量，但规范化需额外标注 / 工程成本，且跨闭源商业产品与所有脚手架架构的外部效度未验证。

---

## 五、给实践的启示（浓缩）

- **不要只报最终准确率**：至少在评测里保留轨迹 / 状态 / 生命周期信号（#7 / #28 / #40 / #46），否则会漏检"成功但违规"或"纪律崩溃"。
- **优先用可执行 oracle，LLM judge 当辅助**：能用引擎 / 状态机 / SQL 校验的就用，LLM judge 务必做 judge 选择分析与复核负担权衡（#6 / #22 / #28 / #32 / #30）。
- **评测结论必须带统计门禁**：报告最小任务预算、噪声地板、harness–model pair，避免单 seed 假显著（#45 / #29 / #36 / #15）。
- **把安全 / 护栏忠实当评测的一等维度**：动作准入、护栏真假拒绝、能力–安全联合，不要等上线再测（#1 / #4 / #9 / #31 / #12）。
- **成本 / 效用要一起报**：可信赖 = 质量 × 可靠性 × 成本，预算感知评测是落地门槛（#5 / #42 / #36）。

---

*附：41 篇测评类论文完整清单见 index.md 表格（标签分类含「测评」者）。本文按主题 1–9 与趋势 / 空白组织，便于按需深入对应笔记。*
