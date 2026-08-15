# IFCMemoryBench: Evaluating Long-Term Memory of LLM-Based Agents in BIM Information Retrieval

## 背景与 Idea
现有 agent 记忆评估大多只测试开放域或 persona 场景下的"对话式回忆"，缺乏在真实结构化、领域特定环境中跨会话复用信息的测试。BIM（建筑信息建模）是一类典型专业工程工作流：agent 必须查询大型 IFC 模型，同时依赖项目规范、客户决策、工程约定等常出现在对话中却不在模型里的信息。作者认为，更强的记忆测试应是"agent 能否在实时、结构化、领域特定的环境中复用先验会话的信息"。为此提出 IFCMemoryBench——一个人工校验的 benchmark，将 IFC-Bench v2 中"信息不完整"的问题转化为多会话记忆任务：把缺失的项目上下文散布在早期会话，再用一个 probe 问题检验 agent 能否结合记忆与实时 IFC 查询作答。

## 核心方法
从 IFC-Bench v2 的 Category 4 问题（答案缺失/模糊/需外部估计）出发，先用三个信息检索系统提取 IFC 侧所用信息，再用 LLM 判定 memory-dependence 标签（not_in_ifc / needs_external_info / partially_answerable），仅保留依赖记忆的任务。随后用带 web 搜索与代码解释器的 LLM agent 合成缺失的外部事实与 gold answer，并用 ReAct agent 回放生成 25–40 个真实感先验会话，把项目知识自然散布其中（含干扰与重引入）。评估框架将记忆分解为 ingestion（写入）、retrieval（检索）、utilization（使用）三阶段，并用专家校验的 LLM judge 同时评测 answer quality（correctness/completeness/relevance 三维度，全满足才算 answer-correct）与 memory quality（retrieval_relevant / covers_key_facts / uses_memory）。比较了向量（Mem0）、时序图（Graphiti）、文件（DeepAgents 的 Markdown，含 cited 变体）三类记忆系统。

## Performance 概括
benchmark 含 143 个多会话任务、19 个项目、23 个 IFC 模型、共 4,016 个先验会话。在部署真实摄取范围（user messages + assistant 最终答案）下，最强系统（Mem0）仅达 32.4% answer accuracy；即便在 oracle-filtered ingestion 或更强 probe agent 下，准确率仍低于 60%。主要瓶颈是 coverage：agent 常取到正确项目与主题的记忆，但遗漏关键事实（最佳系统仅覆盖约一半所需事实）。no-memory 基线答不出任何任务，证实 benchmark 确实依赖跨会话记忆。人类专家对 40 个抽样任务的 judge 校验显示 answer 一致性 95%（κ=0.90）、memory 一致性 100%。

## 论文价值与不足
价值在于首次在"工具驾驶的专业工作流"中系统评估 LLM agent 长期记忆，揭示明显的"领域迁移鸿沟"——通用记忆系统（向量/图/文件）常检索到主题相关但碎片化、不完整的项目知识，提示可靠专业 agent 需要连接对话、项目知识与结构化模型实体的领域感知记忆表示。不足在于：benchmark 数据（项目上下文）为合成或公开来源、未做真实 BIM 部署校准；ingestion scope 最佳结果来自只摄取 user messages，这受数据集构造影响（耐久事实只播种在 user 消息中）；全实验仅用 Grok-4.3 单一模型，跨模型泛化未验证。

## 工业界落地可行性评估
论文面向 Nemetschek/建筑工程（AEC）领域的 BIM 信息检索，这是真实工程需求，且明确指出"领域感知记忆表示"对可靠专业 agent 至关重要，落地场景明确。但当前系统准确率过低（最强 <60%），尚不能直接部署，需领域专属记忆架构与可追溯引用机制（实验中 cited Markdown memory 表现更好）方可落地。
