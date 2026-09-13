# LoRA SFT

## 目的

基础模型能够自然对话，但无法稳定遵循 ShopSimulator 动作协议。监督微调用于建立基础
策略：发出合法工具调用、依据观察收集证据、选择商品规格并正确终止。

## 输入

- 基础模型：`Qwen/Qwen3.5-2B`
- 主数据：`data/sft_pure_v4/all.jsonl`（1,192 行）
- 固定课程清单：`data/sft_curriculum/manifest.json`
- 梯度数据 1,073 行；开发集 119 行；与最终评测集重叠数为 0
- 训练目标：只计算 assistant token；屏蔽 user 和工具观察 token

源数据与标签哈希、精确任务 ID、阶段定义和仅供人工复核的标记，都冻结在课程清单中。
较旧的 `data/sft/` 划分只用于复现历史基线。

## 执行

完成 `bash scripts/setup.sh` 后：

```bash
# 不加载模型，只检查全部六条训练/合并命令。
bash scripts/sft_curriculum.sh --dry-run

# 在服务器上依次执行 A → B → C。
bash scripts/sft_curriculum.sh --swanlab
```

启动器先训练 LoRA adapter，再将其与基础模型合并：

```text
outputs/models/sft-curriculum/stage-a/{adapter,merged}/
outputs/models/sft-curriculum/stage-b/{adapter,merged}/
outputs/models/sft-curriculum/stage-c/{adapter,merged}/
```

默认配方：

| 配置 | 数值 |
|---|---|
| 最大序列长度 | 24,576 |
| Epoch | 每阶段 1 个 |
| 单设备 batch size | 1 |
| 梯度累积 | 8 |
| 学习率 | `1e-4` → `7e-5` → `5e-5` |
| LoRA rank / alpha / dropout | 16 / 32 / 0.05 |
| 梯度 checkpoint | 开启 |
| Attention 实现 | SDPA |
| 保存的 epoch checkpoint | 3 |

长上下文是有意设计：单条训练样本包含完整多轮交互。缩短上下文可能截断终局决策，或
截断支持该决策的证据。

阶段 A 使用 256 条 foundation 数据学习动作协议。阶段 B 从 A 的 merged checkpoint
重新创建 LoRA，并使用 799 条累积 constraint 数据。阶段 C 从 B 以相同方式继续，使用
全部 1,073 条训练数据。因此简单技能训练三遍，约束处理两遍，长程策略一遍。A 完成后
可使用 `--start-stage b` 继续，也可用 `--stop-after-stage b` 限定一次服务器任务范围。
阶段内中断时，使用
`--start-stage <stage> --resume-from-checkpoint <checkpoint-dir>` 恢复。

## 评测

```bash
bash scripts/serve_model.sh outputs/models/sft-curriculum/stage-c/merged
bash scripts/evaluate.sh sft
```

验证损失只反映训练健康度，不是最终模型分数。应依据 119 条开发集及失败类型覆盖选择
阶段。只有在配方冻结后才能运行 Final-200 Clean，避免将最终基准暗中用于 checkpoint
选择。

## 输出合同

GRPO 必须从合并模型开始，不能直接使用 adapter：

```text
GRPO_MODEL_PATH=outputs/models/sft-curriculum/stage-c/merged
```

这个边界让 GRPO 启动器不依赖 SFT 训练进程。
