# 目录说明：`.`

## 本目录职责

- **shenzhi 仓库根**：单仓 monorepo 顶层；聚合 `apps`（可运行服务）、`packages`（共享契约）、`data`（本地/开发用数据，勿提交业务机密）、`infra`（容器与观测样例）、`scripts`（运维与灌库）、`docs`（需求与架构）。

## 子目录导航

- `docs/`：人类与开发 Agent 优先阅读的产品与工程上下文。
- `apps/web`：三页界面与流式、埋点。
- `apps/api`：四 API、Session 编排、主/子 Agent、多轮召回、规则与持久化。
- `packages/shared`：跨端 DTO/常量/schema。
- 其余见各子目录内 `agent.md`。
