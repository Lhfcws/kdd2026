# Comparative Analysis of Agent Evaluation Frameworks: Stability, Detection, and Discovery in Enterprise Analytics Agents

## 背景与 Idea
企业在生产环境部署 LLM 分析 agent（生成 SQL、调用工具、综合答案）时，需要能跨复杂多工具工作流可靠检测失败的评估框架。论文指出大多数团队依赖的临时抽查常漏掉"语法正确但语义错误"的静默失败——本研究显示人工抽查漏掉了 72 个真实失败中的 9 个（12.5%）。为此，论文对三种代表不同评估范式的框架做了系统比较，回答三个研究问题：评估稳定性（RQ1）、已知失败检测（RQ2）、未知风险发现（RQ3）。

## 核心方法
研究采用三阶段比较，在冻结执行轨迹（frozen execution traces）上对两个零售供应链预测分析 agent（Agent A 品类级需求、Agent B 入库前置期，各 9 个工具）进行评估。三个框架为：Strands Evals（基于 OpenTelemetry 完整执行轨迹的 trace-based LLM judge）、PromptFoo（text-only LLM judge 加红队）、Agenta（确定性启发式，零 LLM 成本）。Phase 1 稳定性用 20 个用例 × 3 个 judge 模型 × 5 次重复，测决策一致性与 within-case 标准差（阈值 ≤0.15）；Phase 2 已知风险检测用 72 个跨 6 类失败的 ground-truth 用例；Phase 3 未知风险发现用 Strands 功能测试生成（每 agent 20 例）加 PromptFoo 对抗红队（每 agent 42–48 例）。所有阶段 judge 统一用 Claude Sonnet 4.6。

## Performance 概括
Phase 1：36 个配置中 35 个（97%）达到稳定性阈值；Agenta 因确定性达 100% 决策一致性但平均分恒为 0，PromptFoo 92%，Strands 82%——更深评估带来更多方差（评估深度与方差呈权衡）。Phase 2：Strands 取得最佳 precision-recall 平衡（F1=0.90，FPR 10.3%），PromptFoo 召回匹配（90.9%）却产生 5× 假警报（FPR 48.7%），Agenta F1=0.629；差距来自依赖执行上下文的维度——tool correctness Strands 89% vs PromptFoo 11%，groundedness Strands 0% FPR vs PromptFoo 52%。评估器还发现 9 个被人工误标为"通过"的语义错误。Phase 3：发现 24 个新颖失败，功能测试与安全红队零重叠（Strands 9 个功能类如 SQL bug、schema 伪造；PromptFoo 15 个安全类如 schema 泄漏、系统提示泄露）。

## 论文价值与不足
三条结论具实践价值：评估器能获取的信息量决定评估质量（trace > text > string）；功能与安全评估互补而非可替代（24 个新失败零重叠，安全产出随 schema 复杂度放大）；LLM 评估器能发现人类审阅遗漏的语义错误。论文据此推荐"双框架策略"：Strands 做功能评估、PromptFoo 做对抗红队安全验证。不足在于：未测试这些框架作为 CI/CD 门控时能否检测真实改进/退化（缺乏受控扰动设计），研究为研究设置而非真实生产部署，且 Agenta 的高精度需大量定制开发、对语义错误失效。

## 工业界落地可行性评估
论文直接面向企业分析 agent 部署。明确建议采用两框架组合——trace-based 评估覆盖功能正确性、对抗红队覆盖安全验证——并以评估器-需求映射表（Table 1）作为适用于任意 agent 的固定结构以降低采用门槛。其发现（如"LLM 评估器可作为第二审阅者捕获语义错误""功能与安全需并行"）对正在构建 agent 评估流水线的团队有立即可用的指导意义；作者也指出下一步应把该工作流打包成可复用 toolkit 并检验 CI/CD 判别力。
