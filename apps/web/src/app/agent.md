# 目录说明：`apps/web/src/app`

## 本目录职责

- **App Router 路由与布局**：`layout.tsx` / `page.tsx`；子路径 `assess`、`result/[id]`、`history` 对应需求中的三页或等价信息架构。

## 与后端约定

- 评估提交、历史、联系团队 等走 `lib/api-client` 中封装；与 `session_id` / `assessment_id` 的路由参数保持一致。
