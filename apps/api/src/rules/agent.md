# 目录说明：`apps/api/src/rules`

## 本目录职责

- **可解释风险规则层**：`engine` 对输入特征输出 **高/中/低**、命中 `rule_id` 与原因码；**最终输出前**建议再跑一遍与 LLM 结果对齐/覆盖。

## 子目录

- `definitions/`：YAML/JSON 或版本化规则包，可 CI 校验 schema。
