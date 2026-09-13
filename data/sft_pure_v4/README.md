# Pure DeepSeek-v4 SFT 数据池

本目录包含当前 DeepSeek-v4 教师数据与外部 `merged` 数据包新增部分去重后的并集。

- `all.jsonl`：1,192 条干净、任务唯一且达到 gold purchase 的轨迹。
- `difficulty_labels.jsonl`：任务固有难度，以及 `deepseek-v4-flash` 轨迹的独立过程复杂度标签。
- `duplicate_report.json`：每个重复任务的数据来源、质量特征以及保留/丢弃记录。
- `metadata.json`：数量、哈希、标注来源和配比可行性。

当前 SFT 配方保留自然的 23.8% / 66.9% / 9.2% 难度分布。由于困难样本较少，如果
强行调整为 30% / 50% / 20%，只会为了比例而丢弃有效数据。确定性划分与累积式训练
阶段见 [`../sft_curriculum/`](../sft_curriculum/README.md)。
