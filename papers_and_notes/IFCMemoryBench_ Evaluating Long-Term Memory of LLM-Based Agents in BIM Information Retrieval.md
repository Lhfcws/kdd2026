# IFCMemoryBench: Evaluating Long-Term Memory of LLM-Based Agents in BIM Information Retrieval

## 背景与 Idea
现有 agent 记忆评估大多只测试开放域或 persona 场景下的"对话式回忆"：agent 能否从早先对话里恢复事实、偏好或过去事件。这类设置有价值，但没充分捕捉专业工作流——在那里被记住的信息必须和领域特定工具、结构化数据、不断演化的项目上下文一起使用。作者认为，更强的记忆测试应是"agent 能否在实时、结构化、领域特定的环境中复用先验会话的信息"。

BIM（建筑信息建模，Building Information Modelling）是一类典型专业工程工作流，提供了具体压力测试。BIM 模型是建筑项目的结构化数字表示，包含墙、梁、空间、系统、材料、数量及关系等带类型对象；其主导交换格式 IFC（Industry Foundation Classes，工业基础类）是一个庞大且语义丰富的面向对象 schema。查询 IFC 需要领域专长与技术能力，近年催生了 LLM-based BIM 信息检索：agent 通过 ReAct 式工具使用检视 IFC 文件（如 Hellin 等发布的 IFC-Bench v2，让 agent 对 IFC 写并执行 Python 做自适应探索）。但现有 BIM 检索系统与 benchmark 大多是无状态的（stateless）：假设每个答案都能仅从 IFC 模型导出，测试独立问答对。真实 BIM 工作流不同——用户数周数月回到同一项目，细化需求、确认规格、记录客户决策、修正早先假设，而这些信息大量不进入 IFC 模型，却可能被后续问题依赖（如 IFC 数量需结合排放因子、成本假设、房间表修正或某次会话提到的工程约定）。可靠 BIM 检索 agent 因此必须结合实时 IFC 查询与长期项目记忆。

这正是 agent 记忆评估的更宽缺口：现有记忆系统用向量库、知识图或持久文件跨会话保留信息（如 MemoryBank/Mem0、Graphiti、Claude Code 的 Markdown AGENT.md、DeepAgents 的文件系统记忆），benchmark 如 LongMemEval、MemoryAgentBench、MemoryArena 评估开放域或 persona 对话中的记忆能力，但都不直接测试"在工具驱动的专业工作流中、记忆需与结构化领域数据结合时是否仍可靠"。在此设置下，检索到主题相关记忆不够——记忆必须保留精确的项目事实、单位、实体引用、假设及其与结构化模型的关系。关键区分是 BIM 的 closed-world（问题完全可由 IFC 回答，直接属性查与数量计算）vs open-world（需 IFC 不含的外部上下文，如项目进度、结构规格、设计者意图）问题；本文 benchmark 针对后者。

为何 BIM 检索难且适合做记忆压力测试：IFC 是一个语义丰富的面向对象 schema，查询它需要领域专长与技术能力，因此催生了 LLM-based BIM 信息检索这一子方向。早期工作（如 BIM-GPT）把任务当作自然语言到查询的翻译；近期趋势转向 ReAct 式 agent——观察每步工具结果再决定下一步查询或遍历，Hellin 等进一步推广为"自适应探索"（agent 对 IFC 写并执行 Python 代码），并发布 IFC-Bench v2。本文正是把 IFC-Bench v2 里"信息不完整"的问题转化为多会话记忆任务。需注意现有 LLM-based BIM 检索系统与 benchmark 大多是无状态的：假设每答案都能仅从 IFC 导出，评独立问答对。而真实 AEC（建筑工程）工作流里，BIM 查询常支撑设计协调、合规检查、客户驱动决策，外部上下文（项目进度、结构规格、设计者意图）往往由用户在先前对话中提供——这正是长期 agentic 记忆该保留、却不在 IFC 中的信息，本文 benchmark 瞄准这个 open-world 体制。

论文借用了认知科学的记忆术语作为工程类比：工作记忆（working memory）是处理当前请求时主动维护的任务上下文（指令、中间状态、近期工具输出）；短期记忆（short-term memory）是单会话内的对话历史；长期记忆（long-term memory）跨会话持久化，又细分为语义记忆（去语境化的事实、约束、偏好）、情景记忆（具体过往交互与事件）、程序记忆（技能与例程）。本研究聚焦跨会话的长期记忆，程序记忆留待未来。现有长期记忆系统的实现可归为三类存储导向设计：向量系统（如 MemoryBank、Mem0，把提取事实/摘要/片段存为嵌入按语义相似度检索）、图系统（如 Graphiti，把记忆表示成随会话增量增长的知识图、支持混合检索）、文件系统（如 Claude Code 的 Markdown AGENT.md、DeepAgents 的文件系统记忆，agent 读写纯文本文件增删改）。这三类正是 benchmark 比较的对象。

## 核心方法
从 IFC-Bench v2 的 Category 4 问题（答案缺失/模糊/需外部估计，即 open-world 缺口）出发构造任务：先用三个信息检索系统提取 IFC 侧所用信息，再用 LLM 判定 memory-dependence 标签（not_in_ifc / needs_external_info / partially_answerable），仅保留依赖记忆的任务。随后用带 web 搜索与代码解释器的 LLM agent 合成缺失的外部事实与 gold answer，并用 ReAct agent 回放生成 25–40 个真实感先验会话，把项目知识自然散布其中（含干扰信息与重引入，模拟真实项目里客户决策被反复修正）。

评估框架将记忆分解为三个阶段能力：ingestion（摄取，先验会话是否以有用形式写入记忆）、retrieval（检索，agent 能否取到所需被记上下文）、utilization（使用，agent 能否把该上下文与 IFC 查询结果结合产出最终答案）。同时用专家校验的 LLM judge 评两类质量：answer quality 按 correctness/completeness/relevance 三维度（三者全满足才算 answer-correct）；memory quality 按 retrieval_relevant（检索内容相关）/retrieval_covers_key_facts（覆盖关键事实）/uses_memory（实际用到记忆）。比较了三类代表系统：向量（Mem0）、时序图（Graphiti）、文件（DeepAgents 的 Markdown，含 cited 变体——带引用来源，实验中表现更好）。

benchmark 含 143 个多会话任务、19 个项目、23 个 IFC 模型、共 4,016 个先验会话。任务设计为：探测时刻 𝑡 问一个依赖记忆的项目问题 𝑄𝑡，此前 agent 已观察先验会话 𝑆1…𝑆𝑡−1，记忆系统 𝑓 据此构建记忆库；探测时 agent 还可经 IFC 查询工具 𝑇IFC 查模型。框架对记忆如何表示/访问不可知，只要求被记项目上下文能在探测时使用。论文还设 full-context oracle（把全部先验 user 消息无损拼入 prompt）作为成本化上界与健全性检查，达 83.2% answer accuracy——但论文论证这不构成对记忆系统的否定：benchmark 可超出任何固定上下文窗口（加会话与合成干扰任意降低相关/无关比），且生产部署跨月、多并发用户累积的历史无法每次全量重载（oracle 在每次查询付出该成本，记忆系统通过前置摄取摊销），瓶颈"浮现正确项目事实"恰好是长上下文 dump 解决不了的。

任务构造 pipeline 是 benchmark 的方法论核心：先从 IFC-Bench v2 的 Category 4 问题（答案缺失/模糊/需外部估计，即 open-world 缺口）出发，用三个信息检索系统提取 IFC 侧所用信息，再用 LLM 判 memory-dependence 标签（not_in_ifc / needs_external_info / partially_answerable），只保留依赖记忆的任务；随后用带 web 搜索与代码解释器的 LLM agent 合成缺失的外部事实与 gold answer，并用 ReAct agent 回放生成 25–40 个真实感先验会话，把项目知识自然散布其中，并刻意加入干扰信息与重引入（模拟真实项目里客户决策被反复修正）。这样每个探测问题只能靠"记住的上下文 + 实时 IFC 查询"结合作答，no-memory 基线答不出任何任务从而证实 benchmark 确实依赖跨会话记忆。在部署真实摄取范围（user messages + assistant 最终答案）下，最自然的是只摄取用户消息，而这受数据集构造影响（耐久事实只播种在 user 轮次），是后续版本要修正的。

## Performance 概括
在部署真实摄取范围（user messages + assistant 最终答案）下，最强系统（Mem0）仅达 32.4% answer accuracy；即便在 oracle-filtered ingestion 或更强 probe agent 下，准确率仍低于 60%。主要瓶颈是 coverage：agent 常取到正确项目与主题的记忆，但遗漏关键事实（最佳系统仅覆盖约一半所需事实）。no-memory 基线答不出任何任务，证实 benchmark 确实依赖跨会话记忆。人类专家对 40 个抽样任务的 judge 校验显示 answer 一致性 95%（κ=0.90）、memory 一致性 100%。full-context oracle 达 83.2%，但 indiscriminate 全量摄取在任务上限（约 167k token/任务）已崩溃，印证选择性存储与检索的必要性。跨模型比较（Grok-4.3 vs Gemini-3.5-Flash 作 probe agent，记忆固定）显示 probe LLM 通过搜索努力与证据核验显著影响 agent 侧检索与记忆使用。

## 论文价值与不足
价值在于首次在"工具驾驶的专业工作流"中系统评估 LLM agent 长期记忆，揭示明显的"领域迁移鸿沟"（domain-transfer gap）——通用记忆系统（向量/图/文件）常检索到主题相关但碎片化、不完整的项目知识，提示可靠专业 agent 需要连接对话、项目知识与结构化模型实体的领域感知记忆表示（domain-aware memory representations）。其把"无状态 BIM 检索问题转化为多会话记忆任务"的方法论、以及同时评 answer quality 与 memory quality 的双轨框架，可迁移到其他工具驱动专业领域。论文把暴露这些局限本身视为核心价值：作为受控研究仪器，它能在低风险设置隔离通用记忆系统丢失项目知识的位点，供社区针对性改进。不足在于：benchmark 数据（项目上下文）为合成或公开来源、未做真实 BIM 部署校准；ingestion scope 最佳结果来自只摄取 user messages，这受数据集构造影响（耐久事实只播种在 user 消息中，削弱生态效度——真实项目里 assistant 回复与工具输出也携带可记内容，未来版本会把耐久事实也播种进 assistant 与工具轮次，使 ingestion-scope 选择成为真正的"选择性记忆"问题，并能评程序记忆）；全实验仅用 Grok-4.3 单一模型族同时充当 probe agent、记忆系统内部 LLM 组件与 judge，存在自偏好偏差风险（作者以锚定人工 gold answer 的固定二元 rubric 与近乎完美的专家重标注一致性缓解，但跨模型评判仍属未来稳健性检查）；procedural memory（可复用 IFC 查询例程）留待未来。这些不足被论文明确当作受控研究仪器的边界，其价值恰在于低风险地隔离通用记忆系统丢失项目知识的位点。

## 工业界落地可行性评估
论文面向 Nemetschek/建筑工程（AEC）领域的 BIM 信息检索，这是真实工程需求，且明确指出"领域感知记忆表示"对可靠专业 agent 至关重要，落地场景明确。但当前系统准确率过低（最强 <60%），尚不能直接部署，需领域专属记忆架构与可追溯引用机制（实验中 cited Markdown memory 表现更好，印证可追溯性价值）。落地路径是：先把 IFCMemoryBench 当作领域记忆系统的诊断基准，隔离 ingestion/representation（上游主瓶颈：项目知识被不完整写入、或被碎片化成语境孤立事实，进而限制 retrieval coverage）与 utilization 各环节，再据此构建连接对话、项目知识与 IFC 实体的记忆表示。论文已论证长上下文模型不能替代选择性记忆（生产历史累积、成本与延迟使其无法全量重载），因此专业 agent 的记忆能力仍是刚需，benchmark 可作为该方向的共享诊断与进度追踪工具。

讨论中把主要瓶颈定位在上游的 ingestion 与 representation：项目知识可能被不完整写入、被反转成"模型中不存在"的发现、或被碎化成孤立事实，而这些写入缺陷进而限制了 retrieval coverage——agent 常能取到主题相关记忆，却未必取到答案所需的完整项目事实。实验中带引用的 Markdown 记忆（cited 变体，记录事实来源）表现更好，印证了"可追溯"对专业记忆的价值。跨模型比较还显示，即便记忆库固定，probe LLM 仍通过搜索努力与证据核验显著影响 agent 侧检索与记忆使用，说明"挑战不在检索更多文本，而在以结构化、可追溯形式保存项目知识，并配以能在实时 IFC 旁使用它的 agent"。论文把这些局限的暴露本身视为研究价值：作为受控研究仪器，它能在低风险设置隔离通用记忆系统丢失项目知识的位点，供社区针对性改进。production 部署累积的历史无法每次全量重载，因此选择性存储与检索是刚需，benchmark 暴露的"浮现正确项目事实"瓶颈正是长上下文 dump 解决不了的。
