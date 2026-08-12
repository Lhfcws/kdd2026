# PRISM: Prompt-Refined In-Context System Modeling for Financial Retrieval

## 背景与 Idea
金融检索场景对系统建模质量要求高，但端到端微调方案成本高、迭代慢。论文探索一种 training-free、低成本的路径：用 prompt 精炼与 in-context 系统建模来提升检索质量，使方法易于部署与复用。

## 核心方法
PRISM（Prompt-Refined In-Context System Modeling）结合 prompt engineering、ICL（in-context learning）与多 agent 协作，对金融检索任务做系统级建模而无需训练参数。方法在 FinAgentBench 等金融检索任务上进行评测，强调以提示与上下文而非微调来逼近强模型表现。

## Performance 概括
在 FinAgentBench 私有榜上 PRISM 获得第三名，最佳 NDCG@5 为 0.71818（Run 19）。它是唯一进入 top-3 的 training-free 方法；排名前两位的方法均依赖 fine-tuning（其一用 65 个 agent 的 memex，另一用三个 MiniLM 微调），说明 PRISM 以显著更低的成本达到了可比效果。

## 论文价值与不足
价值在于证明 training-free 方法可以媲美微调方案，部署成本低、迭代快，对资源受限的金融检索团队友好。不足在于仅在单一金融榜单上验证，跨领域、跨检索分布的泛化性尚未充分检验，结论外推需谨慎。

## 工业界落地可行性评估
落地性强：金融检索/研报检索等场景可直接采用 PRISM，无需训练基础设施与标注管线，性价比高。对希望快速上线、控制成本的团队尤其合适。
