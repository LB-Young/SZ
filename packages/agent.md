# 目录说明：`packages`

## 本目录职责

- **工作区内可发布的 npm/pip 子包**；`shared` 为前后端/脚本共享 DTO 与字段常量，**禁止**引用 `apps/api` 或 `apps/web` 源码（单向依赖由 apps 指入 packages）。
