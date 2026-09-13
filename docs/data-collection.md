# 数据采集

## 目标

SFT 阶段需要完整展示购物 Agent 正确使用工具的样本：搜索、打开商品、查看证据、选择
规格，并以有效购买结束。仓库只保留通过验收的纯动作轨迹，不保留历史失败采集记录。

## 数据如何生成

当前采集使用 ShopSimulator Environment v2.1、Reward v3，并以
`deepseek-v4-flash` 作为教师模型，共生成 2,498 条原始轨迹。采集期间，每条轨迹的
动作都实际在 ShopSimulator 中执行。只有 Environment v2.1 返回有效的 Reward v3
gold purchase 时才接收该结果；不会再由第二个模型判断轨迹是否成功。

采集审计：

| 项目 | 数值 |
|---|---:|
| 原始轨迹 | 2,498 |
| 接收轨迹 | 1,026 |
| 接收率 | 41.07% |
| 冻结使用行数 | 1,000 |
| 未使用的合格行 | 26 |

冻结子集划分为 800 条训练数据和 200 条验证数据。两者在任务层面互斥，并且与 GRPO
和 Final-200 评测集的任务 ID 均为零重叠。

## 冻结产物

| 文件 | 行数 | SHA-256 |
|---|---:|---|
| `data/sft/train.jsonl` | 800 | `8c3a6ff0033f6ea672af609891e747d60652ddc17e8d3c8eacb19e9d96dd9477` |
| `data/sft/validation.jsonl` | 200 | `9525cc2fb04a1d8d38ae2db959397da908dde3fea766f580fdcf77d1239533cc` |

原始教师响应有意不提交到仓库；其采集路径保留在 `data/sft/metadata.json` 中，作为
来源记录。

## 重新采集

启动 ShopSimulator，配置兼容 OpenAI 协议的教师模型端点，然后执行：

```bash
export OPENAI_BASE_URL=https://your-provider.example/v1
export OPENAI_API_KEY=your-key

python scripts/collect_sft_data.py \
  --tasks data/grpo/train.jsonl \
  --output-dir outputs/sft-collection \
  --model deepseek-v4-flash \
  --target-accepted 1000 \
  --workers 4
```

`raw.jsonl` 是支持续跑的唯一事实来源。重复运行同一命令会跳过已完成任务，并重建所有
派生产物：

```text
outputs/sft-collection/
  raw.jsonl           完整教师响应与环境结果
  accepted.jsonl      严格通过 Reward v3 的 gold 轨迹
  rejected.jsonl      任务 ID 与确定性拒绝原因
  reject_stats.json   接收情况汇总审计
  sft.jsonl           划分前的脱敏训练行
  train.jsonl         任务互斥的训练集
  validation.jsonl    任务互斥的验证集
  metadata.json       行数、配置与 SHA-256 校验值
```

采集前，命令会排除 `data/evaluation/tasks.jsonl` 中的全部任务 ID；构建产物时还会再次
检查。每个任务最多保留一条合格轨迹。若只需从已有原始数据重建派生文件，无需访问
教师模型或环境：

```bash
python scripts/collect_sft_data.py \
  --build-only \
  --output-dir outputs/sft-collection
```

完成采集审计后，只将 `train.jsonl`、`validation.jsonl` 及对应元数据复制到
`data/sft/`。原始教师响应继续保留在 `outputs/`，不得提交。

## 训练行包含什么

每条 JSONL 数据都是一段对话轨迹，包括：

- 购物需求；
- assistant 的工具调用；
- ShopSimulator 工具观察；
- 最终终局动作；
- 将该行绑定到 Environment v2.1 和 Reward v3 的元数据。

SFT 时会屏蔽 user 和 tool token，只对 assistant 动作计算损失。具体训练配方见
[SFT](sft.md)。
