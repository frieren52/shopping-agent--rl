# 使用 veRL 进行 GRPO

## 目的

SFT 负责教授动作格式并提供可靠的初始策略。GRPO 随后在 ShopSimulator 中采样新轨迹，
利用终局 Reward v3 信号进行优化。目标是在不训练奖励模型的情况下，提高约束满足能力
和终止行为质量。

## 集成边界

veRL 使用固定版本 `verl==0.8.0` 安装，本仓库不复制 veRL 源码。项目自有的集成代码为：

```text
src/shopping_grpo/training/grpo/
  adapter/              AgentLoop 与 ShopSimulator 工具
  compat.py             最小范围的运行时兼容钩子
  dynamic_sampling.py   有界的非零奖励动态采样
```

`scripts/setup.sh` 会应用一个经过 SHA-256 校验的补丁，将有界动态采样器接入 veRL 0.8.0。
如果检测到未知 veRL 版本，安装过程会直接失败，不会继续修改。

## 输入

- 初始策略：`outputs/models/sft-merged`
- 训练集：`data/grpo/train.parquet`（1,000 个任务）
- 验证集：`data/grpo/validation.parquet`（50 个任务）
- 环境：ShopSimulator Environment v2.1
- 奖励：Reward v3

各文件哈希记录在 [`data/grpo/metadata.json`](../data/grpo/metadata.json)。

## 执行

先检查解析后的完整命令：

```bash
bash scripts/grpo.sh --dry-run
```

开始训练：

```bash
bash scripts/grpo.sh
```

重要默认值：

| 配置 | 数值 |
|---|---|
| 算法 | GRPO |
| 每个 prompt 的 rollout 数 | 4 |
| Rollout temperature / top-p | 0.7 / 0.9 |
| 训练 / 验证 batch | 2 / 2 |
| 策略学习率 | `1e-6` |
| LoRA rank / alpha | 16 / 32 |
| 最大模型长度 | 24,576 |
| 最大训练步数 | 500 |
| 保存 / 验证间隔 | 50 / 50 |
| KL reward / KL loss | 关闭 / 关闭 |
| 策略熵测量 | 开启，仅记录 |

动态采样最多生成三个 batch 来寻找可用更新，并且最多允许连续跳过十次更新。这些边界
可以防止奖励完全相同的 batch 造成无限重采样。

每次运行还会在输出目录追加 `training_diagnostics.jsonl`。
`generation_batch` 记录包含所有生成轨迹、公开工具序列、终局结果、奖励拆分、Guard
拒绝原因以及分组保留/丢弃决定。`optimizer_step` 记录保留 veRL 标量指标，包括熵、
PPO KL、clip 比例、响应长度和有效分组率。`skipped_update` 会显示零信号尝试，即使它们
不推进优化器步数。

唯一配置位于 [`configs/grpo.yaml`](../configs/grpo.yaml)。高级覆盖项可以放在 `--` 后：

```bash
bash scripts/grpo.sh -- \
  trainer.total_training_steps=20 \
  trainer.save_freq=10
```

## 导出

评测服务不能直接加载 veRL checkpoint，需要先导出选定 actor：

```bash
bash scripts/export_grpo.sh \
  outputs/models/grpo/global_step_100/actor \
  outputs/models/grpo-merged
```

项目报告使用 step 100。Checkpoint 应依据验证指标选择，不应默认最后一步就是最佳结果。
