# StableEval Arena: A Cost-Aware Agentic Benchmark for Stablecoin Price Stability Prediction

## 背景与 Idea
稳定币是交易、借贷、支付、结算等数字金融基础设施，其价值取决于"锚定一美元"这一脆弱承诺。与传统加密币预测不同，稳定币风险评估不是预测完整价格路径，而是判断其在固定窗口内是否稳定、进入预警或发生锚定压力（peg stress）。随着 agentic AI 进入金融决策支持，仅看最终答案准确率已不够，还需衡量协议遵从、输出合法性、运行时成本与失败模式。论文指出，现有基准未直接测试 agentic 系统在成本与可靠性约束下能否充当稳定币风险监测 agent。其 idea 是把"可信赖性"定义为预测质量、运行可靠性与计算成本的联合属性，用防泄漏（leakage-safe）的历史回放框架，在统一协议下比较 agent 与简单基线，并显式报告稀有压力事件检测、校准、结构化输出合法性与推理成本。

## 核心方法
StableEval Arena 每个案例给定冻结的 30 天（720 小时）回看窗口，要求系统输出结构化的未来 7 天风险评估：stable/watch/stress 三分类、depeg 概率、最大锚定偏离（基点）、方向、置信度与理由，并以严格 JSON 模式约束。标签由冻结评估器按偏离阈值规则生成，隐藏的未来价格与标签仅用于案例选取与评分、绝不进入 prompt。资产覆盖 USDT/USDC/DAI，BTC/ETH 仅作上下文。框架用两套互补实验块：StableEval-120-Enriched（54 stable/45 watch/21 stress，富集压力以做稀有风险诊断）与 StableEval-507-Natural（441/45/21，自然分布全池扩展）。统一后端为 OpenRouter 上的 qwen-2.5-72b-instruct（temperature 0、禁用联网），对比四个原生/框架级 agent（Nanobot、LangGraph、Hermes AI、EvoAgentX）与两个适配型 -Claw 扩展（OpenClaw、NanoClaw），并以 always-stable、historical persistence、threshold rule、simple ML 四类非 agent 基线防止"只有 agent"的比较。指标含 Macro-F1、平衡准确率、stress recall、missed-depeg rate、Brier、MAE/RMSE、valid-JSON 率、失败率、延迟、token 与每案成本。

## Performance 概括
120 例富集块中，historical persistence 基线最强（Macro-F1 0.285、平衡准确率 0.300）；原生 agent 里 Nanobot 最佳（准确率 0.442、Macro-F1 0.351、平衡准确率 0.393），6 个 agent 中 5 个 Macro-F1 超过历史基线，但最高校准首过 stress recall 仅 0.095（历史基线 0.333）。507 例自然分布块中，always-stable 基线准确率 0.870 但 watch/stress recall 全为 0；最强基线 simple ML/logistic 为 Macro-F1 0.402、平衡准确率 0.493、stress recall 0.381；没有任何 agent 在 Macro-F1 上超过该基线（LangGraph 最高 0.304、NanoClaw 0.304、Hermes AI 0.301）。507 池 21 个真实 stress 案例中 6 个系统共漏掉 19 个，持续 depeg 漏检率无 agent 低于 1.0000；而所有 agent 在 120 与 507 上 valid-JSON 率均为 1.0、失败率 0.0，验证器层把最佳 stress recall 提到 0.143 但成本几乎翻倍（Nanobot 从 $0.345 升至 $0.978）。核心结论是：执行可靠性高，但稀有压力与持续 depeg 检测仍很弱。

## 论文价值与不足
价值在于把"过程可靠性 vs 金融风险可靠性"的落差作为稳定币 agent 评测的核心发现，并以双块设计（富集 vs 自然分布）避免两类对称误读，同时把成本、延迟、校准、稀有事件诊断纳入可信赖性视角，且开源数据集与代码、保留完整复现元数据，方法论对金融 agent 评测很有示范意义。不足在于：真实 stress 案例稀少（全池仅 21 个），stress recall 与漏检率应作风险信号而非统计结论；证据包有意有限（仅价格/成交量/市场结构），不含储备证明、赎回摩擦、预言机、跨链流动性等更丰富信号；统一后端 qwen-2.5-72b 不测试后端泛化性；OpenClaw/NanoClaw 仅为适配执行而非原生 CMDOP；少量真实名称/日期带来残余参数记忆风险（匿名化消融显示最大标签变更率 0.333）。

## 工业界落地可行性评估
该基准明确面向稳定币支付基础设施的 agentic 监测层，工业落地场景清晰：论文直接讨论了三类产业启示——委托授权与可互操作性的定期支付、SLA/清算/经济最终性、结算定价与税务问责，指出 agent 可帮助决定执行/暂停/改路由支付，但前提是 stress recall 显著高于当前水平。落地障碍在于稀有压力检测过弱，尚不能支撑自动化支付执行；且其仅覆盖价格稳定性，未来需扩展到流动性、赎回、预言机、监管事件等多模态证据与错失压力惩罚的代价函数。整体是一项诊断型、可审计的评测基础设施，适合作为稳定币风险监测 agent 的回归基准与上线前压力测试，而非即时生产部署依据。
