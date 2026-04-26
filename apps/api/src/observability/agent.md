# 目录说明：`apps/api/src/observability`

## 本目录职责

- **可观测性**：`events` 发射五必选事件 + 可选 `recall_round_*`；`tracing` OpenTelemetry/ span 与 log 关联；**日志脱敏**不输出完整患者叙述到第三方。

## 与需求

- `Architecture-Design.md` §2.4 五事件、§6.4 扩展事件。
