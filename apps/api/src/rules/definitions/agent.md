# 目录说明：`apps/api/src/rules/definitions`

## 本目录职责

- **规则源文件**（可版本子目录如 `v1/`）：与 `rule_id` 一一可映射；**变更需**迁移说明与审计线兼容。

## 与 DB

- `rule_hits` 存引用 id；本目录为「定义真相来源」或导入中间层。
