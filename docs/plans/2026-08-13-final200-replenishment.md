# Final-200 补齐实施计划

**目标：**发布一份包含 200 个任务、经过审计并与训练数据隔离的基准，且不重新引入
任何已拒绝的源任务。

**架构：**沿用现有评测路径和包内盲测 ID Guard。移除所有目标不可达、gold 标注矛盾
或价格要求未评分的源任务；从冻结的 ShopSimulator 池中按约束分层选择替代任务，并在
原子更新清单、Guard 和文档前完成验证。

**技术栈：**Python 标准库、ShopSimulator goal 数据、JSONL/JSON、`unittest`、
Markdown、Git。

---

### 任务 1：定义并测试替代合同

**文件：**

- 修改：`tests/test_evaluation_dataset.py`
- 修改：`data/evaluation/tasks.jsonl`
- 修改：`data/evaluation/metadata.json`
- 修改：`src/shopping_grpo/resources/blind_final_task_ids.json`
- 修改：`src/shopping_grpo/resources/blind_guard.json`

1. 增加回归断言：200 个唯一任务 ID、永久排除 17 个已拒绝 ID、清单哈希匹配、盲测 Guard 一致。
2. 从所有当前训练池之外选择已审计候选 ID，并保持被移除任务的约束数量分布。
3. 验证每个候选任务的目标规格与目标商品一致、明确预算不低于可用目标规格价格，以及指令和标注一致。
4. 同步更新唯一任务列表、元数据和包内 Guard。

### 任务 2：公开选择证据

**文件：**

- 修改：`docs/evaluation-dataset.md`
- 修改：`docs/evaluation-updates.md`
- 修改：`README.md`
- 修改：`data/README.md`
- 修改：`docs/README.md`
- 修改：`docs/evaluation.md`

1. 记录全部替代任务，包括任务 ID、通俗查询摘要和已通过的检查。
2. 说明 Final-200 质量合同及剩余局限：人工审计可发现标签/解析器矛盾，但不能证明每个有效替代商品都会被完全等价地奖励。
3. 将分母变化记录为基准版本变化，不能把旧 Final-183 重算结果重新标记为新 rollout。

### 任务 3：验证与发布

**文件：**验证以上全部改动。

1. 运行聚焦的数据集、Guard 和 benchmark 测试。
2. 独立运行替代任务审计和全部训练/评测重叠检查。
3. 审阅暂存差异，只提交相关文件并推送 `main`。
