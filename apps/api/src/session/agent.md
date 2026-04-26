# 目录说明：`apps/api/src/session`

## 本目录职责

- **Session 编排**：`orchestrator` 单写队列、与连接绑定的 `session_id`；`fsm` 查询级状态与五事件；`context_loader` 装配热上下文/压缩产物；`storage` 读写全量 `messages`/`tool_calls` 到 `data/sessions`。

## 与架构文档对应

- §4 数据流、§5 状态机、§8–§9 压缩与全量落盘。
