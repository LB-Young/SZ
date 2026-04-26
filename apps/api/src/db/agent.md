# 目录说明：`apps/api/src/db`

## 本目录职责

- **关系型持久化**：`models`（ORM）、`session_factory`、可选 `repositories`；**迁移脚本**在 `migrations/`，不混在本目录的 Python 业务里。

## 表职责概览

- 见 `Architecture-Design.md` §11：assessments、advices、evidences、rule_hits、event_logs、经验池、contact_requests 等。
