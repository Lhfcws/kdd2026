# A Fraud-Detection-Inspired Framework for LLM Agents Security

## 背景与 Idea
LLM agent 的自主性带来新攻击面：直接提示注入、间接内容攻击、多轮升级策略。现有防御多聚焦单轮提示级过滤或规则护栏，难以捕捉跨交互序列逐步升级的风险。作者借鉴支付/登录滥用中的成熟欺诈检测，把 agent 安全重新建模为"交互轨迹级对抗检测"——不再判断单条 prompt 是否恶意，而是对交互轨迹建模风险。

## 核心方法
定义五组行为信号（prompt 内容、session 历史、tool 使用、执行上下文、跨轮交互），实例化为结构化特征：prompt（长度/关键词/越权尝试）、session（重试/语义漂移/工具多样性）、tool（one-hot 工具 + task-tool mismatch）、context（外部/敏感内容）、trajectory（累积风险、burst、novelty、context-exfil gap）。用轻量 XGBoost（180 estimators、max_depth=4、42 特征）做二分类，在敏感执行前给风险分并分级决策（allow / restrict / block）。在合成多轮交互语料（12,000 条，60/20/20 切分，测试集 1,200 良性 + 1,200 对抗）上评估，对比 Rule-filter、Prompt-only、Single-step、Seq-window 四个基线（后三者均用 Qwen3-4B）。

## Performance 概括
XGBoost 检测器 F1=0.92、AUC=0.97、ASR reduction 0.90，单线程 CPU 推理仅 4.6ms（占 planner 延迟预算 0.6%）；比 LLM 基线快 9.4–9.5×（Qwen 基线约 43ms），且 F1 最高。消融显示 trajectory 特征单独即可接近全模型（isolated AUC 0.91），在 leave-one-out 中 dominate 其他组，证实跨轮信号是主要贡献。攻击家族包括 split_exfil、context_laundering、privilege_drift、staged_burst。

## 论文价值与不足
价值在于把成熟欺诈检测思路引入 agent 安全，提供低延迟、可解释、与 LLM 护栏互补的轨迹级检测层，实验表明跨轮信号主导性能且速度远优于 LLM 检测器。不足在于实验基于参数化合成模板（非真实 agent 轨迹），缺乏公开多轮对抗 benchmark；特征与攻击家族为作者设计，真实攻击多样性可能未被覆盖；XGBoost 实例化需手工特征工程，泛化到其他 agent 框架需适配。

## 工业界落地可行性评估
框架明确面向实时 agent 防御（在敏感执行前介入），低延迟使其可嵌入生产 agent 管线作为预执行风险信号，与 sandbox / 权限控制 / 人工审查互补，落地可行性高；但需用真实数据校准阈值与特征后再部署。
