# 目录说明：`apps/api/src/core`

## 本目录职责

- **横切基础**：`config`（含库 URL、数据根目录、`K_max` 等）、`logging`（结构化 JSON、`trace_id` 与 `session_id` 注入）。

## 原则

- 无业务 if/else；不依赖 `routes` 反向引用。
