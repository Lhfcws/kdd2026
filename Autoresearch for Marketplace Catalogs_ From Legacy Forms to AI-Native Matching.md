# Autoresearch for Marketplace Catalogs: From Legacy Forms to AI-Native Matching

## 背景与 Idea
双边服务市场正从确定性请求表单（request-form）转向 AI 原生的概率匹配，需要重新生成支撑匹配/搜索/定价的 provider 偏好分类法。Thumbtack 等平台的隐式目录原先散布在数百个类目专属 Q&A schema 中。作者将问题定义为"目录重建"而非新建（目录已隐式存在于 legacy 系统中），并提出以每职业（occupation）为单位的自研究循环（autoresearch loop）逐职业生成偏好标签。该系统已自 2026 年 4 月在生产中部署于 132 个职业。

## 核心方法
每个职业独立运行 propose-evaluate-keep 循环：generator（GPT-5-4）生成候选标签集，六维度 LLM-as-judge（GPT-5-4-mini）打分（/15），7 个 persona critic 并行给出加权惩罚（无硬否决），editor 针对最弱维度单点修改 prompt。关键设计：generator/judge 用 OpenAI 家族，critic/editor/parity mapper 用 Claude Sonnet 4.6（跨家族避免偏见互相强化）；用 parity mapping 把新标签映射回 legacy Q&A，既产生覆盖率信号又提供人工 QA 接口；强制人工 sign-off 才上线。六 rubric 包括 Screening vs Intake、Tag Legibility、Preference Variance、Cross-Category Consistency、Canonical Coverage、Information Loss，并针对产品经历评审做了 recalibration。

## Performance 概括
重建保真度：73.3% 的 legacy 答案直接映射到新标签，20.8% 为 intake-only（尺寸/范围/"灵活"等合理非标签），仅 6.0% 为遗憾遗漏；按应映射项算 pooled coverage 92.5%，职业中位数 100%（均值 97.8%）。生产数据：14 天队列（1,840 个 MVP pro，9.3M 次 filter evaluation）。关键发现——40.66% 的 filter 失败源于"缺失 canonical category tag"，这是部署侧播种缺口而非目录质量问题；Handyman 单独占约 70% filter 事件。循环在迭代 3 前即达最优（best kept tag set 中位数迭代 2），E 分数中位数 13.84/15，循环相对单发生成提升中位数 +2.24 分、改进 119/132 职业。

## 论文价值与不足
价值在于首个生产级目录重建自研究系统，提出 per-occupation 独立循环、marketplace-grounded 加权 critic（无硬否决，避免单 persona 卡死循环）、legacy Q&A parity mapping 等可复用设计，并诚实区分"目录质量"与"部署结果"两类指标（如 40.66% filter 失败是部署侧问题）。不足在于：六 rubric judge 与人类质量的一致性仍需独立外部 evaluator 验证（计划未做）；跨职业标签名漂移（如 Client preference/type 同义异名、大小写不一致）需下一波修复；与 legacy form 的结果级因果对比是未来工作。

## 工业界落地可行性评估
已在 Thumbtack 真实生产部署（132 职业、按需数小时 onboarding 新职业、强制人工 sign-off），是本文最明确的工业落地案例之一，落地可行性高且已被真实流量验证。
