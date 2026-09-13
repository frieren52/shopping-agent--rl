# Final-183 整理实施计划

**目标：**用一份公平、具备泄漏保护且可公开核验的 Final-183 任务列表，替换审计前的
Final-200 基准。

**架构：**继续使用既有 `data/evaluation/tasks.jsonl` 路径，避免为评测调用方增加兼容
参数。以原子方式同步更新元数据和包内盲测 ID Guard，再用 Markdown 记录排除策略和
归档级模型分析。

**技术栈：**JSONL、JSON 元数据、Python `unittest`、Markdown、Git。

---

### 任务 1：锁定整理后的任务合同

**文件：**

- 修改：`data/evaluation/tasks.jsonl`
- 修改：`data/evaluation/metadata.json`
- 修改：`src/shopping_grpo/resources/blind_final_task_ids.json`
- 修改：`src/shopping_grpo/resources/blind_guard.json`
- 修改：`src/shopping_grpo/evaluation/blind_guard.py`
- 测试：`tests/test_evaluation_dataset.py`

1. 增加回归测试，检查 183 个唯一 ID、17 个排除项、匹配的 SHA-256 元数据和一致的包内盲测 ID。
2. 运行测试，确认它在原 Final-200 数据上失败。
3. 只从唯一任务列表中移除审计确认的 ID，并同步更新哈希、元数据和 Guard 资源。
4. 重新运行回归测试和已有盲测 Guard 测试。

### 任务 2：公开整理依据和归档分析

**文件：**

- 新建：`docs/evaluation-dataset.md`
- 新建：`docs/evaluation-updates.md`
- 修改：`docs/README.md`
- 修改：`README.md`
- 修改：`data/README.md`

1. 将 Final-183 记录为当时唯一有效基准，列出每个排除项及原因。
2. 增加只追加的更新格式，记录 Final-200 轨迹审计、逐模型结果、共同 bad case 和 Final-183 重算结果。
3. 将 Final-200 报告标为历史版本，避免混用新旧分母。

### 任务 3：验证与发布

**文件：**验证以上全部改动。

1. 运行聚焦测试以及哈希/重叠检查。
2. 审阅暂存差异，避免混入用户的其他改动。
3. 只提交本次数据整理、文档和已经确认的报告自动化改动，并推送 `main`。
