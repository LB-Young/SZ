# 目录说明：`apps/api/src/migrations`

## 本目录职责

- **数据库 schema 版本**：Alembic 修订（或 ORM 等价物）；**上线顺序**与 `rules/definitions` 及种子脚本协调。

## 环境

- 仅对「库」不对此目录下的 `data/sessions` 文件；后者由 `session/storage` 管理。
