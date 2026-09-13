# Pure V4 SFT 课程训练

`manifest.json` 是当前 SFT 配方唯一的训练清单。它固定 Pure V4、难度标签和评测任务
文件的 SHA-256，记录所有训练/开发集 `task_id`，并保证三个阶段可复现。

| 阶段 | 包含的数据桶 | 训练 | 开发 | Epoch | 学习率 |
|---|---|---:|---:|---:|---:|
| A | foundation | 256 | 28 | 1 | `1e-4` |
| B | foundation + constraints | 799 | 88 | 1 | `7e-5` |
| C | 全部数据桶 | 1,073 | 119 | 1 | `5e-5` |

## 课程权重审计

Pure V4 文件本身不存在某个任务被复制 100 次的问题：共有 1,192 行，对应 1,192 个
唯一任务 ID。合并审计从 258 个重复任务组中删除了 258 行，最终文件中没有发现完全
相同的用户提示。详见 `../sft_pure_v4/metadata.json` 与
`../sft_pure_v4/duplicate_report.json`。

但由于每个阶段都从上一阶段合并后的 checkpoint 开始并训练一个 epoch，课程本身会
形成隐式重加权：

| 数据桶 | 唯一训练行 | 参与阶段 | 每个任务的有效曝光次数 |
|---|---:|---|---:|
| foundation（全部简单任务） | 256 | A、B、C | 3 |
| constraints（全部中等任务） | 543 | B、C | 2 |
| strategy（174 个中等、100 个困难任务） | 274 | C | 1 |

因此实际产生 2,128 次有效样本曝光，而不是 1,073 次。原始训练分布是 23.9% 简单、
66.8% 中等、9.3% 困难；累积曝光后约为 36.1% / 59.2% / 4.7%。结合各阶段学习率，
foundation、constraints 和 strategy 的名义单任务学习率曝光分别为 `2.2e-4`、
`1.2e-4` 和 `5e-5`。由于 Adam 状态和 checkpoint 会变化，这并不等同于精确的优化器
权重，但确实会偏向基础任务、弱化困难策略任务。

这可能解释课程训练收益偏弱：最困难样本只在最后阶段出现一次，而较简单样本被反复
学习。当前结论只是待验证假设，并非因果结论；修改课程前，应与单遍基线对比或记录
逐任务曝光次数。不要直接复制困难样本，否则只会以另一种方式引入人为重加权。

修改任一源文件后，重新生成并审计清单：

```bash
.venv/bin/python scripts/prepare_sft_curriculum.py
git diff -- data/sft_curriculum/manifest.json
```

服务器执行方式：

```bash
bash scripts/setup.sh
bash scripts/sft_curriculum.sh --dry-run
bash scripts/sft_curriculum.sh --swanlab
```

如服务器使用外部虚拟环境，可设置 `SFT_PYTHON=/path/to/venv/bin/python`，不需要修改脚本。

从已完成阶段继续：

```bash
bash scripts/sft_curriculum.sh --start-stage b --swanlab
```

恢复被中断的阶段时，指定对应 Transformers checkpoint：

```bash
bash scripts/sft_curriculum.sh \
  --start-stage b \
  --resume-from-checkpoint outputs/models/sft-curriculum/stage-b/adapter/checkpoint-100 \
  --swanlab
```

`review_flags` 只是人工分流清单，不代表自动删除。搜索次数多或轨迹较长的样本可能正是
有价值的困难样本。Final-200 Clean 必须从所有梯度数据中排除，也不得用于调整这些阶段。
GRPO 的初始模型为：

```text
outputs/models/sft-curriculum/stage-c/merged
```
