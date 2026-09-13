# 训练诊断实施计划

**目标：**在不改变学习目标的前提下，保留足够的 SFT 与 GRPO 证据，用于比较 checkpoint
并诊断强化学习收益偏低的问题。

**设计：**复用 Transformers checkpoint 日志和现有 veRL/Shopping 指标。为每条
rollout 增加公开动作与 Guard 摘要，将生成分组和优化器指标追加到一个 JSONL 文件，
开启熵测量，并按默认配置为每个 SFT epoch 保留一个 checkpoint。

## 任务

1. 为 rollout 诊断、JSONL 追加行为、启动器接线、熵配置和 SFT checkpoint 保留编写 CPU 测试。
2. 实现最小的项目侧 JSONL 辅助模块，并暴露缺失的 rollout 字段。
3. 修改固定版本 veRL 补丁，使其在 GRPO 输出旁写入生成事件与优化器事件。
4. 更新默认值和文档，运行聚焦 CPU 检查；如本地存在固定版本 veRL 源码，再验证补丁。
