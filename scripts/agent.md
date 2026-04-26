# 目录说明：`scripts`

## 本目录职责

- **维护与数据脚本**（可 Python/Node/shell）：规则导入、知识灌库、冒烟测试；**不**在 CI 中硬编码生产连接串。

## 与 packages

- 可依赖 `packages/shared` 的常量，避免与 `apps` 业务代码重复。
