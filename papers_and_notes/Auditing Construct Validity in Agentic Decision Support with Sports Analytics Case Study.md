# Auditing Construct Validity in Agentic Decision Support with Sports Analytics Case Study

## 背景与 Idea
多数 agent benchmark 只问"任务是否完成"，但一类新兴系统产出的是给人类决策者的"分析性断言"（临床决策支持、组合风险归因、足球战术分析报告）。这类输出没有 oracle、无法用准确率评判，正确问题更接近"领域专家是否会辩护这个论断"。近期 Brookings–CMU–NIST 研讨会呼吁把测量科学中的"构念效度（construct validity）"引入 agentic AI 评估，但未操作化。作者提出 TRACE——一个四威胁审计框架，将构念效度落地到 agent 产出的分析性论断，每个威胁配一个可度量审计工具，并在足球战术分析上实例化为诊断性 benchmark PressureBench。

## 核心方法
四个威胁：T1 构念误设（agent 操作了错误目标）、T2 判定不可靠（多 judge 不一致）、T3 跨运行随机不稳定、T4 模型升级回归。对应工具：APCR（反模式捕获率）、IAJA（多 judge 一致性，Cohen's/Fleiss' κ）、SRI（科学可复现指数，5 次重跑的 formula/citation/verdict Jaccard 与 verdict 精确匹配）、MERS（模型演化回归分，跨版本 ΔF1）。PressureBench 含 38 条防守压迫分析提案：24 条植入反模式（E1–E6 各 4 条，均锚定已发表足球分析文献）、6 条有效基线、4 条 hard distractor（表面像反模式实则合理）、4 条由 Opus 4.7 无偏生成以破除自评分担忧。统一事件日志记录每次 LLM 调用，支持 replay。

## Performance 概括
四个部署相关发现：(i) 能力层级 ≠ 精度代理——最小模型 Haiku 4.5 在有效锚点上 FP=0.17（95% CI [0.00,0.50]），最大 Opus 4.7 达 0.67 [0.33,1.00]，hard distractor 上 0.00 vs 0.75（CI 不重叠）；(ii) 单次 producer 调用不安全——N=5 重跑中仅 17% 得到相同审计 verdict，且换 Sonnet 4.6 复现同一 0.17；(iii) 每反模式回归差异大——E1 ΔF1=0（evergreen），E3 达 0.35（regression-prone）；(iv) 对抗式 producer 压力测试（避用触发词）中审计管线仍捕获 10/12=83%。简单 rule 层在 6 个反模式中 4 个上胜过 LLM judge（如 E1 F1=0.89）；整轮评估 276 次 LLM 调用仅花费 $10.90。

## 论文价值与不足
价值在于把测量科学中的构念效度首次端到端操作化为"框架 + benchmark + 多 judge 校准 + 重跑协议 + 模型演化监控"，并公开全部数据/脚本/日志；识别了"单次 producer 调用不安全""小模型 FP 更低"等反直觉的部署结论。不足在于：仅足球一个领域、仅 Anthropic Claude 一家模型，跨域跨厂商泛化未验证；每反模式 n=4，点估计尚属初步；原子单元是单条论断而非完整交付物；引用伪造、战术幻觉等失败模式需扩展本体接地与文献核查才能覆盖。

## 工业界落地可行性评估
论文明确给出"工业管线"形态（producer → 四审计工具 → 审计标注报告 → 主教练），主张与 Deloitte 等现有监控平台互补而非竞争，直接面向足球战术分析等工业决策支持场景，落地可行性高；但当前范围窄，真实落地需跨域验证与监管级（可篡改证据）日志加固。
