# 目录说明：`packages/shared`

## 本目录职责

- **跨端共享契约**：TypeScript/JSON Schema 或 Pydantic 模型（二选一或双生成）；`types`、`constants`、`schemas` 子目录分职责。

## 发布

- 可 `workspace:*` 引用；锁版本在根 lockfile 管理。
