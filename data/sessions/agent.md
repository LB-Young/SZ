# 目录说明：`data/sessions`

## 本目录职责

- **Session 冷存储**（架构 §9）：按 `session_id` 子目录放 `messages.jsonl`、`tool_calls.jsonl`、压缩快照等；**权威全量**供审计与子 Agent，非 PostgreSQL 主表。

## 与 LLM 上下文

- 进模型的「热上下文」为压缩后的装配结果；**本处保留不可变/版本快照**。
