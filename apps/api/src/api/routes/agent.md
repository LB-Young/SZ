# 目录说明：`apps/api/src/api/routes`

## 本目录职责

- **四组路由实现**：`assessments`（提交流式、按 id 取结果）、`history`（列表）、`contact`（协同请求）。  
- 入参出参 DTO 与 `packages/shared` 对齐；流式用 `SSE` 或分块 JSON Lines。

## 禁止

- 在 route 内直接写 SQL/向量检索，应经 repository 与 retrieval 层。
