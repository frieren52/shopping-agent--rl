# 实验产物

本目录保存 README 结果表背后体积小、可复核的实验产物。大型 checkpoint 和完整轨迹
不会存入 Git。

```text
baseline/   基础模型评测配置与摘要
sft/        SFT 训练/评测配置与摘要
grpo/       GRPO 训练/评测配置与摘要
comparison.md
```

所有报告模型均使用相同的 200 个留出任务、Environment v2.1 和 Reward v3。结果解释
与协议限制见 [comparison.md](comparison.md)。
