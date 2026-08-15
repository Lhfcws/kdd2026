# How Much Coordination Gain Is Real? A Paired Noise-Floor Protocol for Multi-Agent LLM Benchmarks

## 背景与 Idea
多智能体 LLM 协调（multi-agent coordination）架构常以"小幅基准提升"作为证据，宣称某一种协调设计优于另一种——例如用共享记忆、结构化 handoff、或服务端状态来让多个 agent 互相通气。但作者 Alibek Kaliyev（UT Austin）与 Artem Maryanskyy（Uber）质疑：当一个协调机制在 trial 0（即协调存储还为空、机制逻辑上尚未激活）时，两协议对 API 的输入配置是等价的，那么它们之间的配对差异究竟有多大？如果连"配置等价、机制未激活"的两组跑出来的配对差距都能到十几个百分点，那么那些被当作"架构胜利"的小幅提升，很可能只是同一量级的测量噪声。

背景里有一个关键事实支撑这种质疑：Multi-Agent System Taxonomy（MAST）对 1600+ 条生产 agent 轨迹的分类显示，跨 agent 协调失败占所有失败的 36.94%，是单一最大类别；这说明协调本身是一个可度量、可研究的真实目标，而"如何可靠地度量协调增益"才是承重问题。作者最初其实是要评估两种协调设计（agent 侧的 pull 通道与框架侧的 intercept 通道）相对无协调基线在 τ²-bench retail 上的表现，结果 head-to-head 在可用功效下完全检测不到效应——正是这个"非结果"逼出了真正的问题：在 API 输入配置等价的协议之间，run-to-run 的方差到底有多大？一个单独的数字就足以翻转对近期大量协调文献的解读。

论文因此提出"协调噪声地板（coordination noise floor）"协议：它要求任何协调架构的性能声明，都必须先跨过一个"在同模型、同输入、配对测量下得到的本地噪声地板"，否则无法区分真实的协调增益与单纯测量噪声。噪声地板不是 universal 常数，而是 local、随模型与域变化的量；它的角色是作为发布门禁（release gate），而非对协调研究本身的否定。

论文的动机来自一个很具体的协调失败场景：两个 LLM agent 并行处理同一条多步行程，第一个尝试订机票、发现无库存后放弃并转向别处，而几秒后由同一 orchestrator 启动的第二个 agent，通过同一 API 尝试同一个预订、重新撞上完全相同的死路——第一个 agent 的失败只活在它私有的 scratchpad 里，第二个无从知晓。这正是当前多 agent LLM 系统（MAS）协调方式的结构性弱点：失败不被共享。近期架构用协调通道（结构化 handoff、共享内存、服务端状态）来修补，并报告小幅基准增益作为"某设计胜过另一设计"的证据；本文要问的正是这些增益里有多少是真的、有多少只是 run-to-run 方差。

## 核心方法
测量基底是 ET-MCP，一个任务作用域（task-scoped）的负面知识（negative-knowledge）事件存储，符合 MCP 2026-07-28 规范，通过 Model Context Protocol 暴露，作为隔离"读者侧选择"的底层设施（作者明确说 ET-MCP 是支撑物而非贡献）。它只记录失败类事件（failed paths、constraint violations、abandoned approaches 等五类，映射到 MAST 失败分类），用 TF-IDF 排序检索同伴失败。两种读者侧架构跑在同一存储上：pull 让 agent 自己当主动读取者，在估计同伴事件可能有用时调用 trace.query 把摘要放进系统提示；intercept 把读取者移到框架层，框架拦截每一次工具调用，当 (name, arguments) 命中存储里的同伴失败事件时，在响应前注入一个 [PEER-WARNING] 块。intercept 复用 Strands、AgentCore Policy、LangChain、Microsoft Agent Framework 已有的 hook 表面，所以是已有运行时的"加法扩展"而非新框架。

协议的核心是"同模型、配对、配置等价（configuration-equivalent）"的方差测量。在 τ²-bench retail（一个状态校验的客服工具使用域，按 Chen et al. 的无偏估计报告 pass_k）上，用 Claude Haiku 4.5 同时担任 assistant 与 user simulator，T=0，20 轮预算，每任务 2 trial，对 no_coord / pull / intercept 三协议做 n=100、双 seed 的配对测量。所谓配置等价，是经代码审查确认两协议在某 trial 的请求侧增强行为一致（系统提示、工具列表、采样参数、消息数组构造相同）；论文通过 SHA-256 字节审计确认 trial 0 请求等价——离线的 client.messages.create 参数重建 40/40 单元字节一致，live 的 user-simulator 打开 30/30 字节一致。注意配置等价是字节一致的必要非充分条件：Messages API 不暴露 seed，T=0 请求并非比特确定，作者用配对 n=30 同协议复现把纯 API 随机性（E2）上界钉在 ≤3 pp。

论文定义了 coordination-active pass_k（协调活跃通过率）：pass_k 只在"协调存储非空"的任务上才计入统计——亦即同伴协调存储有内容、协调机制逻辑上确实激活的那些任务。这与边际 pass_k（marginal pass_k，回答"是否该部署整个系统"）不同：coordination-active pass_k 回答"当协调有机会发挥作用时，协调是否真的帮了忙"。因为空存储 trial 上的跨协议差异测到的只是噪声而非架构，所以这个条件化指标正是为了把 E2–E4 类噪声从真实机制效应中剥离。论文把它作为最小发布门禁，并给出三个运行时告警（R1 对 coordination-active pass_k 做 CUSUM、R2 滚动 95 分位注入事件数、R3 协调键冲突率）以在 Amazon Bedrock AgentCore / SageMaker Model Monitor 上落地。

统计方法上，效应量统一用 Cliff's δ，配对用 sign test（McNemar 等价且已报告），多重比较校正统一用 Bonferroni（m=3）。Haiku n=100 retail 的 head-to-head 是预先注册的主测试（协议、K=3 writer、seed 42、T=0 在跑前冻结）；pass_k_active 虽因果可预先指定，但是应用在已冻结数据上。整体设计强调"同模型、同输入下的配对比较"，以隔离协调机制本身的边际贡献，而非跨模型跨域的混合对比。

指标与试验结构需要先说清。τ²-bench 按 Chen et al. 的无偏估计报告 pass_k：k=1 是边际成功率，k=2 是两 trial 都成功的率；completion rate 则是最终输出通过 benchmark scorer 的比例。每个协议在测试时跑 n=100 任务、每任务 2 trial（trial 0 与 trial 1），其中 trial 0 时协调存储为空、机制逻辑上未激活，trial 1 时存储已含同伴事件、协调机制逻辑上激活——这正是"协调活跃子集"的来源（coordination-active pass_k 在 k=2 下等价于"trial 0 失败的任务上 trial 1 的成功率"）。trial-0 的配对差距被分解为四层来源以诚实地呈现 headline 数字：E1 是字节一致的首请求审计、E2 是协议内 API 随机性（≤3pp）、E3 是用户模拟器发出首个采样输出后的协议间漂移、E4 是假想的 pull 特定结构扰动（第二 seed 未复现，已撤回）。严格说，trial 0 测到的是"两个配置等价 harness 运行之间的多轮配对方差"，之所以称其为协调噪声地板，是因为它是该 benchmark 上任何协调声明必须跨过的门禁，而非方差本身具有协调特异性。

机制诊断给出三种朴素协调的失败模式（preliminary）：M1 写入错归因——因果相关的调用常在上游，而 last-K 只记录下游症状（单法官 Haiku-4.5 审计 30 个可分析 trial-0 失败，77% 指向上游，但属方向性动机而非确认效应）；M2 逐次注入噪声——intercept 存储平均每 trial 累积 2.77 个事件（最多 8），每次命中的调用都触发一次独立警告；M3 脆弱键接——字面元组负载不迁移，cancel_order(#W123) 匹配不到 cancel_order(#W456) 即使两者都指向已发货订单。作者据此给出三条匹配现有生产 hook 表面的改进（P1 因果归因写入、P2 选择性拦截、P3 谓词约束提取），并额外实现 P0 oracle 正控制（注入 golden action 序列）以验证协议确实能检测真实增益——结果 oracle 在 n=35 下仍 p=0.79 不显著，说明该基准的协调活跃子集本身可能非知识受限，多数失败并非"缺知识"而是用户模拟器动态所致，强化了"负结论只针对朴素协调"的读法。

## Performance 概括
trial 0 的干净配置等价对比（no_coord vs intercept）在两 seed 合并下为 +5pp（Wilson 95% CI [-2, +12]，不显著）；最大单 seed 对比 +18pp（pull vs intercept，seed-1，p_corr=0.012）在第二 seed 未复现（−3pp，p_corr=1.0）；合并后 +7.5pp，Wilson 上界 ≈15pp。双 seed 配对差距包络为 [-3, +18]pp，合并上界 Wilson CI ≲15pp。任何单 seed 的 Bonferroni 显著在测到第二 seed 后都消失。7/10 近期多 agent 协调架构报道的提升低于该本地噪声地板，另有 1 篇落在包络内（12–15pp 带），其余 2 篇高于 15pp。协调活跃子集（informative pairs 仅 8–17）在当前功效下无可检测效应（pull/intercept 在 trial-1 都坍缩到 0.54 成功率），属于"功效不足下的非结果"而非干净的零效应。跨模型/域探针显示地板随能力与域变化：Haiku airline（n=30）差距坍缩到 ≈0pp，Sonnet retail（n=30）方向翻转且幅度 3.3–20pp。样本量表（Table 3）显示 τ²-bench retail 公开 114 任务、单 seed 只能检测 Δ≥20pp（独立臂）或 Δ≥15pp（配对），远不够分辨小幅增益。

## 论文价值与不足
价值在于首次给出"同模型配对噪声地板"作为协调架构声明的发布门禁，并给出了一个可操作的指标（coordination-active pass_k）与三个运行时告警。最具说服力的实例是作者自己的 seed-1 pull-vs-intercept 达 p_corr=0.012 看似显著，却在第二 seed 直接翻转（−3pp）——这正是该协议设计要抓的"单 seed 假阳性"。论文把近期 10 篇协调架构的 headline 增益与之并列，指出 7 篇低于本地地板、1 篇落在区间内，直观暴露了"跨设置单数字对比"的方法论风险。它也诚实区分了"对朴素协调的否定"与"对协调研究本身的否定"：负面结论只适用于朴素 last-K 写入 + 朴素读取，且作者给出 P1–P3 三条机制层面的改进方向（因果归因写入、选择性拦截、谓词约束提取）预期能跨过该地板。

不足主要来自规模与代表性。主结论仅限 Haiku 4.5 + retail 域的 n=100 双 seed；跨模型/域只是 n=30 的探索性探针，明确说 Sonnet n≥250 才是确认性测试。协调活跃子集样本过小（8–17 个 informative pairs），head-to-head 是"功效不足下的非结果"，不是确定性零效应；作者自己做的 oracle 正控制（注入 golden action）也只在 n=35 下得到 p=0.79，说明该基准的协调活跃子集本身可能不是知识受限的、难以分离真实增益。局限还包括：配置等价靠代码审查 + 40/40 SHA-256 审计，n=100 主扫描未逐字节审计（但审计证明请求组装不变式成立）；writer 的 K=3 是在先验 n=30 上调的；M1（写入错归因）的 77% 上游率来自同模型类单法官判断，无 Cohen's κ 或独立复现；ET-MCP 假设可信单租户部署，共享 trace 存储带来投毒写入与跨主体泄露的安全面。

## 工业界落地可行性评估
中等可行，且定位清晰：它不是一个要训练的新模型，而是一套"上线前同模型配对地板 + coordination-active pass_k 门禁 + 运行时告警"的评估/监控协议，可直接用于多 agent 系统的发布前评估与线上漂移监控。其运行时告警（CUSUM / R2 / R3）明确面向 Amazon Bedrock AgentCore 与 SageMaker Model Monitor 这类平台，intercept 机制又复用已有 hook 表面，工程改造成本低。但它不能直接"套用"：噪声地板是 local 而非universal，随模型能力层与业务域剧烈变化（Haiku airline ≈0pp、Sonnet retail 方向翻转），因此每个团队必须用自己的模型与业务域重跑配对协议，得到域相关的 MDE（最小可检测效应，Table 3 给出了样本量规划）。论文给出的实用规则是：任何协调声明都附一份同模型配对探针（baseline vs 惰性 hook 基线，n=30 即可），只有当宣称效应在探针上界约 2 倍内时才升级到完整 n≥100 协议。落地障碍在于：需要可复现的成对 harness（作者已开源并绕过 τ²-bench/litellm 的已知 bug）、需要双 seed 以排除单 seed 假阳性、且要把"协调活跃子集是否知识受限"作为基准设计的前提——否则即便完美协调内容也可能无法在子集上分离出显著增益。论文的 Table 3 已给出样本量规划（MDE 与所需 n 的对应表），团队可据此决定在自己的模型/域上要跑多少才能分辨宣称的增益；跨模型/域只做了 n=30 的探索性探针（明确说 Sonnet retail 需要 n≥250 才是确认性测试），因此不要把地板当作通用常数直接迁移。总的判断是：噪声地板协议最适合作为"协调架构发布前的 CI 门禁"与"线上协调漂移监控"，而非用于证明某个协调设计的压倒性胜出。
