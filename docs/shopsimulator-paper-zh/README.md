# ShopSimulator 原论文中文资料

本目录整理阿里团队论文 **ShopSimulator: Evaluating and Exploring RL-Driven LLM Agent for Shopping Assistants**（arXiv:2601.18225）。

## 文件

- [中文精读版](shopsimulator-paper-zh.md)：按原论文结构整理的中文译述，包含公式、主要表格、实验结论、局限与附录要点。
- [浏览版](shopsimulator-paper-zh.html)：与相邻项目笔记一致的暖色文章排版，可直接在浏览器中阅读。
- [英文原论文](original/shopsimulator-arxiv-2601.18225.pdf)：下载自 arXiv 的原始 PDF。

## 一页结论

ShopSimulator 是面向中文电商的购物 Agent 训练与评测环境。它包含约 **134 万件淘宝商品、12 个领域和 28,147 个任务**，同时覆盖单轮、多轮、个性化单轮和个性化多轮四类场景。实验使用高相似商品组成的 Catalog-Fine，以强化对型号、属性和规格的细粒度辨别；论文构建段写“约 2 万件”，统计段按任务覆盖口径写“24K”，中文精读版保留了这一口径差异。

最重要的结果有五点：

1. 现有强模型仍不可靠。GPT-5 的四场景平均完全成功率 `R_succ` 为 **32.65%**，单轮为 **40.78%**，个性化多轮仅 **24.13%**。
2. 最大难点不是找到大致相关的商品，而是在长轨迹中持续满足全部属性、规格和价格约束。
3. SFT 与 RL 互补。对 Qwen3-8B，SFT 主要注入操作流程先验，RL 进一步改善个性化偏好利用和细粒度匹配。
4. `SFT + GRPO + 严格乘法奖励` 在四类场景都取得最佳结果；Qwen3-8B 的完全成功率分别达到 **38.89%、57.33%、35.50%、34.35%**。
5. 严格乘法奖励优于宽松加法奖励，因为它具有“短板效应”：任一关键约束不满足都会显著压低总奖励，使训练更关注属性和规格这些弱项。

## 与本仓库的关系

论文覆盖四类交互场景，而本仓库遵循自身的 Environment v2.1、Reward v3、observation v2 与 tool schema v2 合约。论文中的结果用于理解环境来源和训练思路，**不能直接当作本仓库当前评测结果**。

## 来源

- arXiv 摘要页：<https://arxiv.org/abs/2601.18225>
- 官方代码与数据：<https://github.com/ShopAgent-Team/ShopSimulator>
- DOI：<https://doi.org/10.48550/arXiv.2601.18225>
