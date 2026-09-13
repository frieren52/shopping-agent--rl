# SFT 数据采集实施计划

**目标：**恢复一条可复现的“教师模型 rollout → SFT 数据”流水线，并与仓库当前的
ShopSimulator 和训练合同保持一致。

**架构：**复用 `evaluation/rollout.py` 完成模型与环境交互。新增一个采集模块，负责
确定性的 Reward v3 验收和 SFT 数据行构建；再提供一个支持续跑的命令行入口，同时写入
原始与派生产物。原始教师响应保留在 `outputs/`，生成训练/验证文件前排除留出评测任务 ID。

**技术栈：**Python 标准库、现有 OpenAI 兼容 rollout 客户端、ShopSimulator
Environment v2.1、Reward v3、`unittest`。

---

### 任务 1：冻结采集合同

**文件：**

- 新建：`tests/test_sft_collection.py`
- 新建：`src/shopping_grpo/collection/__init__.py`
- 新建：`src/shopping_grpo/collection/sft.py`

1. 为严格 Reward v3 验收、纯动作脱敏、留出任务排除和稳定的任务级划分编写失败测试。
2. 运行 `env PYTHONPATH=src python3 -m unittest tests.test_sft_collection`，确认因导入不存在而失败。
3. 使用当前 `environment.actions`、`environment.tools` 和 `training.sft.dataset` 接口完成最小采集模块。
4. 重新运行聚焦测试并确认通过。

### 任务 2：增加支持续跑的采集命令

**文件：**

- 新建：`tests/test_collect_sft_data_cli.py`
- 新建：`scripts/collect_sft_data.py`

1. 为批次路径、达到目标接收数后停止、worker 参数传递和安全默认值编写失败测试。
2. 运行聚焦 CLI 测试，确认命令尚不存在。
3. 实现单一 CLI：从 `raw.jsonl` 恢复，可选并发采集，然后重建 accepted、rejected、SFT、训练、验证和元数据产物。
4. 重新运行两个聚焦测试文件。

### 任务 3：文档与验证

**文件：**

- 修改：`docs/data-collection.md`
- 修改：`README.md`

1. 记录可执行命令、输出结构和验收规则。
2. 在不访问模型或 ShopSimulator 的情况下，运行采集、rollout、SFT 数据和 CLI 单元测试。
3. 运行语法与空白检查，审阅最终差异后提交并推送 `main`。
