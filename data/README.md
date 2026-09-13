# 数据说明

本目录只保留当前项目流程实际使用的数据集。

| 阶段 | 文件 | 行数 |
|---|---|---:|
| SFT | `sft/train.jsonl`、`sft/validation.jsonl` | 800 / 200 |
| GRPO | `grpo/train.parquet`、`grpo/validation.parquet` | 1000 / 50 |
| 评测 | `evaluation/tasks.jsonl`（Final-200 Clean） | 200 |

相邻的 `metadata.json` 记录 SHA-256 校验值和数据来源。SFT、GRPO 与评测划分在任务
层面完全隔离。生成的轨迹必须放在 `outputs/`，不能写入 `data/`。如需重新采集 SFT
数据，应先使用 `scripts/collect_sft_data.py` 生成并审计新数据，再将确认后的训练集和
验证集提升到本目录。
