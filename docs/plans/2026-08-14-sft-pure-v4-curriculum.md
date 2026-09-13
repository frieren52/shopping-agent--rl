# SFT Pure V4 课程训练实施计划

**目标：**将 Pure V4 设为经过审计的 SFT 数据源，并提供一套可通过单条服务器命令
端到端执行的确定性三阶段课程训练。

**架构：**准备脚本验证全部源数据，生成按技能分层的固定训练/开发清单，并记录只供
人工复核的过程标记。现有 SFT 训练器增加 task ID 过滤；轻量课程运行器依次训练并合并
累积阶段 A、B、C，不复制 26 MB 的轨迹文件。

**技术栈：**Python 标准库、现有 Transformers/PEFT 启动器、JSON、Bash、`unittest`。

---

### 任务 1：生成确定性课程清单

**文件：**

- 新建：`scripts/prepare_sft_curriculum.py`
- 新建：`tests/test_prepare_sft_curriculum.py`
- 新建：`data/sft_curriculum/manifest.json`

**步骤 1：**为严格验证、稳定数据桶划分、评测集零重叠、累积阶段数量和复核标记编写失败测试。

**步骤 2：**运行 `python -m unittest tests.test_prepare_sft_curriculum -v`，确认构建器尚不存在并导致导入或断言失败。

**步骤 3：**使用标准库实现构建器：读取 Pure V4 数据与标签，验证质量合同，按带种子的 task 哈希划分原子数据桶，并写出包含源文件哈希和任务 ID 的单一清单。

**步骤 4：**运行聚焦测试，并根据仓库数据生成需要提交的清单。验证总数
`1192 = 1073 train + 119 development`，以及数据桶数量 `256/28`、`543/60`、`274/31`。

### 任务 2：按课程 ID 过滤源数据

**文件：**

- 修改：`src/shopping_grpo/training/sft/dataset.py`
- 修改：`scripts/train_lora_sft.py`
- 修改：`tests/test_sft_training.py`
- 修改：`tests/test_train_lora_sft_cli.py`

**步骤 1：**编写失败测试，证明请求的 task ID 会在分词前完成过滤，且 CLI 接受一个清单和阶段选择器。

**步骤 2：**运行聚焦测试，确认因 API/参数缺失而失败。

**步骤 3：**为加载器增加可选 task ID 过滤，并增加清单/阶段 CLI 参数。两者均未提供时保持原有行为。

**步骤 4：**运行全部 SFT 数据集和 CLI 测试。

### 任务 3：运行并合并三个累积阶段

**文件：**

- 新建：`scripts/run_sft_curriculum.py`
- 新建：`scripts/sft_curriculum.sh`
- 新建：`tests/test_sft_curriculum.py`

**步骤 1：**为清单展开、A/B/C 命令构建、上一阶段 merged 模型传递、dry-run 行为和起止阶段验证编写失败测试。

**步骤 2：**运行测试，确认运行器尚不存在并导致失败。

**步骤 3：**实现轻量 subprocess 编排器，调用现有训练器和合并器，不复制模型加载或训练逻辑。

**步骤 4：**确认 dry-run 输出全部六条命令，最终合并路径为
`outputs/models/sft-curriculum/stage-c/merged`。

### 任务 4：记录服务器直接使用方式

**文件：**

- 修改：`docs/sft.md`
- 修改：`data/sft_pure_v4/README.md`
- 新建：`data/sft_curriculum/README.md`

**步骤 1：**记录单命令启动、阶段大小、输出、恢复示例、监控参数、最终 GRPO checkpoint，以及不使用 Final-200 选择 checkpoint 的原因。

**步骤 2：**使用 `--help` 或 `--dry-run` 输出核对每条命令。

### 任务 5：验证项目包

**文件：**验证以上全部文件。

**步骤 1：**重新生成清单，并要求 Git 差异为零。

**步骤 2：**运行聚焦课程测试和现有 SFT/Guard 测试。

**步骤 3：**运行 Python 编译、Shell 语法、`git diff --check` 和服务器 dry-run。

**步骤 4：**审阅最终差异，只提交课程训练相关文件。
