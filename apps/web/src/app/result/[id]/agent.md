# 目录说明：`apps/web/src/app/result/[id]`

## 本目录职责

- **单条结果详情**（动态路由 `id` = assessment 或业务主键）：展示规则引用、生成时间、知识/经验引用；触发 `result_viewed` 埋点。

## 注意

- URL 中 id 与后端 DTO 字段对齐；避免在 URL 中暴露可枚举的敏感信息（必要时用不透明 token）。
