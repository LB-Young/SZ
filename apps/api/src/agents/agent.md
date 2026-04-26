# 目录说明：`apps/api/src/agents`

## 本目录职责

- **主 Agent 编排**、**子 Agent**（落库/经验）、**compression** 入口（可调用 `session` 存储写快照）。  
- `main` 内注册 tools 与多轮循环；`subagents` 在 **一轮 user query 结束**后异步触发。

## 文件约定

- `compression.py`（或同职责模块）：上下文超过 50% 时摘要策略，**压缩前**全量已写入 `session/storage`。
