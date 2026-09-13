# ShopSimulator

本目录是项目内嵌的 ShopSimulator 环境快照，包括商品归档、Environment v2.1、Reward v3
以及结构化 `/api/shop_agent` 服务。上游源码提交记录在
[`EMBEDDED_SOURCE.json`](EMBEDDED_SOURCE.json)。

不建议手动安装或启动本目录。请在仓库根目录执行：

```bash
bash scripts/setup.sh
bash scripts/start_environment.sh
```

生成的商品 JSON、搜索索引、虚拟环境和日志均不得提交。
