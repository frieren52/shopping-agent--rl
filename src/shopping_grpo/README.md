# 包结构

Python 包按照项目工作流组织：

```text
environment/       连接 ShopSimulator，并执行动作协议
training/sft/      构建只对 assistant token 计算损失的 SFT 样本
training/grpo/     将购物 AgentLoop 和奖励接入 veRL
evaluation/        硬规则检查、Rubric 整理、轨迹 Judge 与结果汇总
cli.py             安装包提供的轻量命令
smoke.py           仅使用 CPU 的公开冒烟测试路径
```

面向用户的命令仍位于仓库根目录的 `scripts/`。这些启动脚本调用本包模块，不构成另一套
重复实现。
