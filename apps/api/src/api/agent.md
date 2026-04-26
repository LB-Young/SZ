# 目录说明：`apps/api/src/api`

## 本目录职责

- **HTTP 边界**：`deps` 注入 DB session、当前用户/匿名 id；**不包含**长耗时 Agent 的完整实现，仅调 `session` / `agents` 门面。

## 子目录

- `routes/`：按资源拆文件，与 OpenAPI 一一对应。
