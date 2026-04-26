# 目录说明：`apps/api`

## 本目录职责

- **BFF 与领域后端**：`HTTP` 四路由、**Session 单写**、**主 Agent 与多轮经验/知识召回**、**子 Agent 落库与经验**、**规则引擎**、**数据库与迁移动**、**可观测**。

## 与数据目录

- Session **全量冷存储**路径由 `core/config` 指向仓库外或 `../data/sessions`（见 `data/sessions/agent.md`）。
