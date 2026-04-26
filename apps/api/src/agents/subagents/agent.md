# 目录说明：`apps/api/src/agents/subagents`

## 本目录职责

- **子 Agent 1 `persist`**：从完整对话抽 `assessment`/`advice`/`evidence`/`rule source`/`event log` 等字段，幂等写库。  
- **子 Agent 2 `experience`**：可复用经验入 **tmp 池**、与持久化池冲突处理、统计晋升（架构 §7）。

## 执行方式

- 建议异步任务队列；失败入重试/死信并打 `event log`。
