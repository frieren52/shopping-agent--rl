# 评测报告自动化实施计划

**目标：**让每个 Shopping GRPO 评测目录都能依据自身的 `summary.json` 和
`trajectories.jsonl` 生成可复用、与具体模型无关的 HTML 报告。

**架构：**为现有自包含报告生成器增加 `--run-dir` 参数，并从 `summary.json` 推导模型
与协议标签。任务级聚合继续由 Python 完成，HTML 只嵌入紧凑数据；常规评测命令结束后
自动调用生成器，同时保留面向历史运行的显式报告命令。

**技术栈：**Python 标准库、现有 HTML/CSS/JavaScript 模板、Bash 封装、`unittest`。

---

### 任务

1. 编写失败测试，证明临时评测目录可被解析，且模型名称会出现在报告数据中。
2. 为 `scripts/build_glm_report.py` 增加 `--run-dir` 和 `--output`，移除硬编码的 GLM-5.2 路径与标签。
3. 增加 `scripts/report.sh NAME`，作为已有输出目录的一条命令入口。
4. 更新 `scripts/evaluate.sh`，使其在评测结束后生成 `report.html`。
5. 运行聚焦测试、现有 benchmark CLI 测试和 Python 编译检查，并重新生成 GLM-5.2 报告验证兼容性。
