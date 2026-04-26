# 目录说明：`apps/api/src/retrieval`

## 本目录职责

- **多轮经验与知识召回**（架构 §6）：`experience` / `knowledge` 两路；`merge` 去重与轮次元数据；`recall_policy` 实现 K_max、早停、覆盖度。  
- **只负责检索与合并**，不直接写业务主表；结果交给 `agents/main` 组进 prompt 与 `evidence` 引用表。
