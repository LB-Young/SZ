# 目录说明：`apps/api/src/db/repositories`

## 本目录职责

- **按聚合的 CRUD/查询**：历史列表、按 assessment 取全量、事件追加；**事务边界**在 service 或 route 的 unit of work 中明确。
