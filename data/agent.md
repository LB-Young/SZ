# 目录说明：`data`

## 本目录职责

- **机本地与开发用数据根**：**不提交**敏感内容到 Git；Session 全量、可选本地向量/缓存。  
- 生产应指向挂载卷或对象存储，路径由 `apps/api` 的 `core/config` 配置。

## .gitignore

- 全目录或除 `agent.md` 与 `.gitkeep` 外应忽略；见根 `.gitignore`。
