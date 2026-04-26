# 目录说明：`apps/api/tests/integration`

## 本目录职责

- **集成测试**：真实 Postgres+向量、或 testcontainers；Session 全量路径指向临时 `data` 子目录。

## 注意

- 不依赖生产 `data/sessions`；用 fixture 建 session 与 assessment。
